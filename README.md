# Add a new field and add validation to an ASP.NET Core MVC app

>This tutorial teaches how to add new fields to entities and migrate them into the database. Then we will learn how to validate properties using `ValidationAttribute`. This guide is compiled based on [Get started with ASP.NET Core MVC](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mvc-app/start-mvc?view=aspnetcore-8.0&tabs=visual-studio-code) by `Microsoft`.

In this section Entity Framework Code First Migrations is used to:

- Add a new field to the model.
- Migrate the new field to the database.

When EF Code First is used to automatically create a database, Code First:

- Adds a table to the database to track the schema of the database.
- Verifies the database is in sync with the model classes it was generated from. If they aren't in sync, EF throws an exception. This makes it easier to find inconsistent database/code issues.

In this validation section:

- Validation logic is added to the Product model.
- You ensure that the validation rules are enforced any time a user creates or edits a product.

Before coming to this guide, please refer to [Get started with ASP.NET Core MVC, Connect to SQL Server database to CRUD](https://github.com/NguyenPhuDuc307/get-started-dotnet-mvc).

## Part 1: Add a new field

- **Add a Rating Property to the Product Model**

  Add a `Rating` property to `Models/Product.cs`:

  ```c#
  using System.ComponentModel.DataAnnotations.Schema;

  namespace MVCProduct.Models;

  public class Product
  {
      public int Id { get; set; }
      public string? Title { get; set; }
      [Column(TypeName = "decimal(18,2)")]
      public decimal Price { get; set; }
      public string? Rating { get; set; }
  }
  ```

  Build the app:

  ```bash
  dotnet build
  ```

  Because you've added a new field to the `Product` class, you need to update the property binding list so this new property will be included. In `ProductsController.cs`, update the [Bind] attribute for both the `Create` and `Edit` action methods to include the Rating property:

  ```c#
  [Bind("Id,Title,Price,Rating")]
  ```

  Update the view templates in order to display, create, and edit the new `Rating` property in the browser view.

  Edit the `/Views/Products/Index.cshtml` file and add a Rating field:

  ```html
  <table class="table">
      <thead>
          <tr>
              <th>
                  @Html.DisplayNameFor(model => model.Title)
              </th>
              <th>
                  @Html.DisplayNameFor(model => model.Price)
              </th>
              <th>
                  @Html.DisplayNameFor(model => model.Rating)
              </th>
              <th></th>
          </tr>
      </thead>
      <tbody>
          @foreach (var item in Model)
          {
              <tr>
                  <td>
                      @Html.DisplayFor(modelItem => item.Title)
                  </td>
                  <td>
                      @Html.DisplayFor(modelItem => item.Price)
                  </td>
                  <td>
                      @Html.DisplayFor(modelItem => item.Rating)
                  </td>
                  <td>
                      <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |
                      <a asp-action="Details" asp-route-id="@item.Id">Details</a> |
                      <a asp-action="Delete" asp-route-id="@item.Id">Delete</a>
                  </td>
              </tr>
          }
      </tbody>
  </table>
  ```

  Update the `/Views/Products/Create.cshtml` with a Rating field:

  ```html
  <form asp-action="Create">
      <div asp-validation-summary="ModelOnly" class="text-danger"></div>
      <div class="form-group">
          <label asp-for="Title" class="control-label"></label>
          <input asp-for="Title" class="form-control" />
          <span asp-validation-for="Title" class="text-danger"></span>
      </div>
      <div class="form-group">
          <label asp-for="Rating" class="control-label"></label>
          <input asp-for="Rating" class="form-control" />
          <span asp-validation-for="Rating" class="text-danger"></span>
      </div>
      <div class="form-group">
          <label asp-for="Price" class="control-label"></label>
          <input asp-for="Price" class="form-control" />
          <span asp-validation-for="Price" class="text-danger"></span>
      </div>
      <div class="form-group">
          <input type="submit" value="Create" class="btn btn-primary" />
      </div>
  </form>
  ```

  Similarly, update the `/Views/Products/Edit.cshtml` with a Rating field:

  Finally run the following .NET CLI commands:

  ```bash
  dotnet ef migrations add AddRatingInProduct
  ```

  ```bash
  dotnet ef database update
  ```

## Part 2: Add validation

- **Add validation rules to the product model**

    The DataAnnotations namespace provides a set of built-in validation attributes that are applied declaratively to a class or property. DataAnnotations also contains formatting attributes like `DataType` that help with formatting and don't provide any validation.

    Update the `Product` class to take advantage of the built-in validation attributes `Required`, `StringLength`, `RegularExpression`, `Range` and the `DataType` formatting attribute.

    ```c#
    using System;
    using System.ComponentModel.DataAnnotations;
    using System.ComponentModel.DataAnnotations.Schema;

    namespace MVCProduct.Models;

    public class Product
    {
        public int Id { get; set; }
        [StringLength(60, MinimumLength = 3)]
        [Required]
        public string? Title { get; set; }
        [Range(1, 100000)]
        [DataType(DataType.Currency)]
        [Column(TypeName = "decimal(18,2)")]
        public decimal Price { get; set; }
        [RegularExpression(@"^[A-Z]+[a-zA-Z0-9""'\s-]*$")]
        [StringLength(20)]
        [Required]
        public string? Rating { get; set; }
    }
    ```

    The validation attributes specify behavior that you want to enforce on the model properties they're applied to:

  - The `Required` and `MinimumLength` attributes indicate that a property must have a value; but nothing prevents a user from entering white space to satisfy this validation.

  - The `RegularExpression` "Rating":

    - Requires that the first character be an uppercase letter.
    - Allows special characters and numbers in subsequent spaces. "PG-13" is valid for a rating.
  - The `Range` attribute constrains a value to within a specified range.

  - The `StringLength` attribute lets you set the maximum length of a string property, and optionally its minimum length.

  - Value types (such as `decimal`, `int`, `float`, `DateTime`) are inherently required and don't need the `[Required]` attribute.
  
- **Validation Error UI**
  
  Run the app and navigate to the `Products` controller.

  Select the **Create New** link to add a new product. Fill out the form with some invalid values. As soon as jQuery client side validation detects the error, it displays an error message.

  ![Validation Error UI](resources/validation-create.png)

Above are all instructions on validate properties using `ValidationAttribute`, refer to the [ValidationAttribute Class](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations.validationattribute?view=net-8.0).

Next let's [Seed the database an ASP.NET Core MVC app](https://github.com/NguyenPhuDuc307/seed-the-database).
