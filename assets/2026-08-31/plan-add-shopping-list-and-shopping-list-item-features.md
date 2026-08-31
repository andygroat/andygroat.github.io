# 🎯 Add Shopping List and Shopping List Item Features

Adds full CRUD for **ShoppingList** aggregates and their **ShoppingListItem** children, following the existing vertical-slice conventions used by the `Todos` feature.

## Design decisions (from clarifications)
- **Soft delete** via `BusinessObjectStatus` — read queries filter out `Deleted`.
- **Nested item routes** under `/api/shoppinglists/{listId}/items`.
- **GET shopping list** returns list only (title, id) — items require a separate call.
- **Update shopping list** = title only. Items managed only through item endpoints.
- **Validation:** list title required, max 100; item title required, max 200.

## Domain (Todo.Vsa.Model/Domain/ShoppingLists)
- `ShoppingList : BusinessObject`
  - `Title` (string, `[Required, MaxLength(100)]`)
  - `ICollection<ShoppingListItem> Items` (navigation)
  - `[Table("ShoppingLists", Schema = Schemas.Default)]`
- `ShoppingListItem : BusinessObject`
  - `Title` (string, `[Required, MaxLength(200)]`)
  - `IsComplete` (bool)
  - `ShoppingListId` (Guid, FK)
  - `[Table("ShoppingListItems", Schema = Schemas.Default)]`

## Persistence (Todo.Vsa.DataAccess/Context/TodoDbContext.cs)
- Add `DbSet<ShoppingList> ShoppingLists` and `DbSet<ShoppingListItem> ShoppingListItems`.
- In `OnModelCreating`, configure one-to-many `ShoppingList → ShoppingListItems` with cascade delete on the FK (only used for EF metadata; deletes are soft).

## Feature slices (Todo.Vsa.Api/Features/ShoppingLists)
Each slice = one file, `public static class` with nested `Command`/`Query` record, `Validator`, internal sealed `Handler` (primary ctor injecting `TodoDbContext`, `ILogger<Handler>`), and a `Map...Endpoint` extension. All handlers return `Result<T>` and endpoints translate via the existing pattern from `CreateTodo.cs`. All read handlers filter `Status != BusinessObjectStatus.Deleted`.

**Shopping list slices:**
1. `CreateShoppingList.cs` — `POST /api/shoppinglists` — returns `Guid`.
2. `GetShoppingLists.cs` — `GET /api/shoppinglists` — returns collection of `{Id, Title}`.
3. `GetShoppingListById.cs` — `GET /api/shoppinglists/{id}` — returns `{Id, Title}` or NotFound.
4. `UpdateShoppingList.cs` — `PUT /api/shoppinglists/{id}` — title only.
5. `DeleteShoppingList.cs` — `DELETE /api/shoppinglists/{id}` — sets `Status = Deleted` on the list AND its items (soft cascade).

**Shopping list item slices (nested):**
6. `CreateShoppingListItem.cs` — `POST /api/shoppinglists/{listId}/items` — validates list exists & not deleted.
7. `GetShoppingListItems.cs` — `GET /api/shoppinglists/{listId}/items` — returns `{Id, Title, IsComplete}[]`.
8. `GetShoppingListItemById.cs` — `GET /api/shoppinglists/{listId}/items/{itemId}`.
9. `UpdateShoppingListItem.cs` — `PUT /api/shoppinglists/{listId}/items/{itemId}` — title + IsComplete.
10. `DeleteShoppingListItem.cs` — `DELETE /api/shoppinglists/{listId}/items/{itemId}` — soft delete.

Endpoints use `.WithName(...)` and `.WithTags("shoppinglists")` / `.WithTags("shoppinglistitems")`.

## Endpoint registration
- Add `Features/ShoppingLists/RegisterShoppingListEndpoints.cs` (mirrors `RegisterTodoEndpoints.cs`) calling all ten `Map...Endpoint` extensions.
- Update `Infrastructure/Extensions/WebApplicationExtensions.MapWebApplication` to call `app.MapShoppingListEndpoints();` after `MapTodoEndpoints()`.
- **No manual MediatR/FluentValidation registration** — handlers/validators auto-register from the API assembly per existing convention.

## Tests (Todo.Vsa.Api.Tests)
Mirror existing test style (TUnit `[Test]` / `[Arguments]` / `await Assert.That(...)`). For each slice, add tests for handler happy path + validator rules. Use in-memory `TodoDbContext` (as existing tests do). Cover:
- Create list — persists with correct title, returns id.
- Get lists/GetById — excludes soft-deleted.
- Update — modifies title, NotFound for missing/deleted.
- Delete — marks list + items as Deleted.
- Item CRUD — validates parent list exists/not deleted; IsComplete toggling via Update.
- Validator tests: empty title, over-length title (list=100, item=200).

## Conventions to honor
- File-scoped namespaces, primary constructors, `sealed` records/classes, `internal` handlers, `public` contracts.
- `CancellationToken` propagated end-to-end.
- Structured Serilog templates (e.g., `"Created ShoppingList {ShoppingListId}"`).
- Use `TypedResults`/`Results.*` — no `IActionResult`.
- Return `Result<T>` from handlers; map to `Results.Created/Ok/NoContent/NotFound/BadRequest` in endpoints (follow patterns already used in the Todos feature; `NotFound` → `Results.NotFound(...)`).

**Progress**: 100% [██████████]

**Last Updated**: 2026-08-27 08:06:27

## 📝 Plan Steps
- ✅ **Create `Todo.Vsa.Model/Domain/ShoppingLists/ShoppingList.cs` — derives from `BusinessObject`, `Title` + `Items` navigation, `[Table]` attribute.**
- ✅ **Create `Todo.Vsa.Model/Domain/ShoppingLists/ShoppingListItem.cs` — derives from `BusinessObject`, `Title`, `IsComplete`, `ShoppingListId`, `[Table]` attribute.**
- ✅ **Update `Todo.Vsa.DataAccess/Context/TodoDbContext.cs` — add `DbSet<ShoppingList>` and `DbSet<ShoppingListItem>`; configure one-to-many relationship in `OnModelCreating`.**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/CreateShoppingList.cs` — Command, Validator (title required, max 100), Handler, `MapCreateShoppingListEndpoint` (`POST /api/shoppinglists`).**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/GetShoppingLists.cs` — Query, Handler filtering out `Deleted`, `MapGetShoppingListsEndpoint` (`GET /api/shoppinglists`).**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/GetShoppingListById.cs` — Query with `Id`, Handler returns NotFound `Result` when missing/deleted, `MapGetShoppingListByIdEndpoint`.**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/UpdateShoppingList.cs` — Command (`Id`, `Title`), Validator, Handler updates title only, `MapUpdateShoppingListEndpoint` (`PUT /api/shoppinglists/{id}`).**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/DeleteShoppingList.cs` — Command, Handler soft-deletes list and all its items, `MapDeleteShoppingListEndpoint` (`DELETE /api/shoppinglists/{id}`).**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/CreateShoppingListItem.cs` — Command (`ListId`, `Title`), Validator (title required, max 200), Handler verifies parent list exists & not deleted, `MapCreateShoppingListItemEndpoint` (`POST /api/shoppinglists/{listId}/items`).**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/GetShoppingListItems.cs` — Query by `ListId`, Handler filters `Deleted`, `MapGetShoppingListItemsEndpoint`.**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/GetShoppingListItemById.cs` — Query, Handler, `MapGetShoppingListItemByIdEndpoint`.**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/UpdateShoppingListItem.cs` — Command (`ListId`, `ItemId`, `Title`, `IsComplete`), Validator, Handler, `MapUpdateShoppingListItemEndpoint` (`PUT /api/shoppinglists/{listId}/items/{itemId}`).**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/DeleteShoppingListItem.cs` — Command, Handler soft-deletes item, `MapDeleteShoppingListItemEndpoint`.**
- ✅ **Create `Todo.Vsa.Api/Features/ShoppingLists/RegisterShoppingListEndpoints.cs` — `MapShoppingListEndpoints` extension invoking all ten `Map...Endpoint` methods.**
- ✅ **Update `Todo.Vsa.Api/Infrastructure/Extensions/WebApplicationExtensions.cs` — call `app.MapShoppingListEndpoints();` from `MapWebApplication`.**
- ✅ **Add TUnit tests in `Todo.Vsa.Api.Tests` under a `ShoppingLists/` folder — one test file per slice covering handler happy paths, NotFound/soft-delete cases, and validator rules.**
- ✅ **Build the solution and run `dotnet test` on `Todo.Vsa.Api.Tests` — fix any compilation or test failures.**

