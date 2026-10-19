# Module structure

## Layout

```
src/
  app.ts                        # createApp(): mounts modules, injects deps, maps errors (the edge)
  modules/
    orders/
      dto/
        create-order.dto.ts     # request schemas + derived types
        order-response.dto.ts   # response shape + mapper
      orders.controller.ts      # validate → call service → shape response
      orders.service.ts         # business rules
      orders.repository.ts      # data access; split into orders.read-repository.ts and
                                # orders.write-repository.ts (and search/stats) when it grows
      orders.errors.ts          # typed domain errors with codes
      orders.rules.ts           # pure rules, unit-tested
      index.ts                  # public surface: the controller and what others may use
  platform/                     # the one seam: env, bindings, current user, language
  integrations/                 # role modules: payments/, email/, ai/, storage/
  db/
    schema.ts
    migrations/
  shared/
    errors.ts                   # DomainError base class, error handler, validate()
test/
  factories/  fakes/  helpers/  fixtures/
  modules/orders/create-order.test.ts
```

Adapt names to the framework (Hono, Express, Fastify, NestJS…), but keep the layers.

## The layers

| Layer | Does | Never does |
|---|---|---|
| Controller | Parses input with the DTO schema, gets the current user, calls the service, maps the result to the response DTO | Business rules, data access |
| Service | Business rules, authorization decisions, orchestration of repositories and role modules | Know about HTTP, parse raw requests |
| Repository | Data access with the ORM; translates constraint violations into domain errors | Business rules |
| DTOs | Schemas for input and output; types derived from them | Hand-written duplicate types |

Naming: a schema is `XSchema` and its derived type is `X` (`CreateOrderBodySchema` → `CreateOrderBody`), on both client and server, so a shared contracts package reads the same everywhere.

## A complete slice

```ts
// db/schema.ts — the invariant lives in the database
export const orders = sqliteTable(
  'orders',
  {
    id: text('id').primaryKey(),
    userId: text('user_id').notNull().references(() => users.id),
    status: text('status', { enum: ['pending', 'paid', 'cancelled'] }).notNull(),
    createdAt: integer('created_at', { mode: 'timestamp_ms' }).notNull(),
  },
  (t) => [
    uniqueIndex('orders_one_pending_per_user').on(t.userId).where(sql`${t.status} = 'pending'`),
    index('orders_user_created_idx').on(t.userId, t.createdAt),
  ],
);
```

```ts
// modules/orders/dto/create-order.dto.ts
import { z } from 'zod';

export const MAX_ITEMS_PER_ORDER = 50;

export const CreateOrderBodySchema = z.object({
  items: z
    .array(z.object({ productId: z.string().min(1), quantity: z.number().int().positive() }))
    .min(1)
    .max(MAX_ITEMS_PER_ORDER),
});
export type CreateOrderBody = z.infer<typeof CreateOrderBodySchema>;
```

```ts
// modules/orders/orders.errors.ts
import { DomainError } from '../../shared/errors';

export class OrderAlreadyOpen extends DomainError {
  constructor() {
    super('ORDER_ALREADY_OPEN');
  }
}
export class OrderNotFound extends DomainError {
  constructor() {
    super('ORDER_NOT_FOUND');
  }
}
```

```ts
// modules/orders/orders.repository.ts
export class OrdersRepository {
  constructor(private readonly db: Database) {}

  findOpenByUser(userId: string) {
    return this.db.query.orders.findFirst({
      where: and(eq(orders.userId, userId), eq(orders.status, 'pending')),
    });
  }

  async insertPending(order: NewOrder, items: NewOrderItem[]) {
    try {
      // One atomic unit (D1 has no interactive transactions: use batch).
      await this.db.batch([this.db.insert(orders).values(order), this.db.insert(orderItems).values(items)]);
    } catch (error) {
      if (isUniqueViolation(error)) throw new OrderAlreadyOpen(); // orders_one_pending_per_user
      throw error;
    }
  }
}
```

```ts
// modules/orders/orders.service.ts
export class OrdersService {
  constructor(
    private readonly repo: OrdersRepository,
    private readonly clock: Clock = systemClock,
  ) {}

  async create(userId: string, input: CreateOrderBody) {
    // Fast path for a clear error; the partial unique index is the real guarantee under concurrency.
    if (await this.repo.findOpenByUser(userId)) throw new OrderAlreadyOpen();

    const order = { id: crypto.randomUUID(), userId, status: 'pending' as const, createdAt: this.clock.now() };
    await this.repo.insertPending(order, toOrderItems(order.id, input.items));
    return order;
  }
}
```

```ts
// modules/orders/orders.controller.ts
export const ordersController = new Hono<AppEnv>().post(
  '/',
  validate('json', CreateOrderBodySchema), // maps schema failures to the shared validation error
  async (c) => {
    const order = await ordersServiceFor(c).create(currentUser(c).id, c.req.valid('json'));
    return c.json(toOrderResponse(order), 201);
  },
);
```

```ts
// modules/orders/index.ts
export const ordersServiceFor = (c: Context<AppEnv>) =>
  new OrdersService(new OrdersRepository(db(c)), c.var.deps.clock);
export { ordersController } from './orders.controller';
```

## The app factory: one place to mount and inject

```ts
// app.ts
export type Deps = { clock: Clock; payments: Payments; email: Email };

export function createApp(overrides: Partial<Deps> = {}) {
  const app = new Hono<AppEnv>();

  app.use(requestId());
  app.use(async (c, next) => {
    c.set('deps', { clock: systemClock, payments: defaultPayments(), email: defaultEmail(), ...overrides });
    await next();
  });

  app.route('/orders', ordersController); // the central router only mounts modules
  app.route('/users', usersController);

  app.onError(handleError);
  app.notFound(handleNotFound);
  return app;
}

export default createApp();
```

Production uses the defaults; tests call `createApp({ payments: fake.payments })` (see `testing.md`). No module mocking.

## One seam to the platform

Bindings, env, the current user and the request language come through small accessors (`env(c)`, `db(c)`, `currentUser(c)`, `language(c)`). Each layer receives only what it needs: the service gets repositories and role modules, never the request context.

## Dependency injection with production defaults

```ts
export class SubscriptionsService {
  constructor(
    private readonly repo: SubscriptionsRepository,
    private readonly payments: Payments = defaultPayments(),
    private readonly clock: Clock = systemClock,
  ) {}
}
```

`defaultPayments()` returns the lazily created, per-instance client, so constructing a service per request never rebuilds it.

## Heavy clients

Build SDK clients, DB connections and AI clients once per runtime instance, lazily on first use; never at import time and never per request.

```ts
let payments: Payments | undefined;
export const defaultPayments = (): Payments => (payments ??= new StripePayments());
```

## Queries

- Every query filters and sorts on indexed columns; the index ships in the same migration as the query that needs it.
- Related rows load with a join or one `inArray` query, never one query per row in a loop (N+1).
- List queries are paginated with a cursor (see `boundaries-and-errors.md`).
