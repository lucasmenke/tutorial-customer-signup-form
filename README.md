# Customer Signup Form

The "Customer Signup Form" allows the customer to sign up for something (f.e custoemr program, raffle etc.). The Webpage is meant to be used on a tablet to get the customers signature.

Finished date of project: 10.12.2022

***

## Tech Stack

- MVC (.NET Framework 4.7.2)
- SQL Database
- JS -> for Signatare Pad

***

## Planing project

<ins>Features</ins>
- capture customer data
- capture customer signature
- store data in a database

***

## Project Structure

1. MVC > has a one sided dependency with the Class Library -> project needs Class Library to build
	1. View -> UI
	2. Controller -> Routing / Frontend Logic
	3. Models -> Frontend Models
2. Class Library
	1. DataAccess -> CRUD Methodes for SQL Database
	2. Models -> Backend Model
	3. BusinessLogic -> Backend Business Logic
3. SQL Server Database
	1. dbo/Tables/Customer.sql -> stores customer data

***

## Extensions

<ins>MVC</ins>
- no extra extensions installed

<ins>Class Library</ins>
-Dapper

***

## Data Structure

- customer
	- Id
	- Salutation
	- Title
	- Surname
	- Forname
	- Street
	- Zip
	- Region
	- Country
	- Mail
	- Phone
	- Smartphone
	- Birthday

***

## SQL

1. mark primary key as identity
``` SQL
CREATE TABLE [dbo].[Customer]
(
	[Id] INT NOT NULL PRIMARY KEY IDENTITY, 
    [Salutation] NCHAR(10) NOT NULL, 
    [Title] NCHAR(10) NULL, 
    [Surname] NVARCHAR(50) NOT NULL, 
    [Forname] NVARCHAR(50) NOT NULL, 
    [Street] NVARCHAR(100) NOT NULL, 
    [Zip] NVARCHAR(50) NOT NULL, 
    [Region] NVARCHAR(50) NOT NULL, 
    [Country] NVARCHAR(50) NOT NULL, 
    [Mail] NVARCHAR(50) NOT NULL, 
    [Phone] NVARCHAR(50) NULL, 
    [Smartphone] NVARCHAR(100) NULL, 
    [Birthday] DATE NOT NULL
)
```

***

## Models

- create Backend Model -> Customer.cs

``` C#
namespace DataLibrary.Models
{
    public class Customer
    {
        public int Id { get; set; }
        public string Salutation { get; set; }
        public string Title { get; set; }
        public string Surname { get; set; }
        public string Forname { get; set; }
        public string Street { get; set; }
        public string Zip { get; set; }
        public string Region { get; set; }
        public string Country { get; set; }
        public string Mail { get; set; }
        public string Phone { get; set; }
        public string Smartphone { get; set; }
        public DateTime Birthday { get; set; }
    }
}
```

***

## Data Access

- create a DataAccess Layer which will be referenced by the BusinessLogic Layer

``` C#
namespace DataLibrary.DataAccess
{
    public static class SqlDataAccess
    {
        // returns the connectin string from MVCApp/Web.config
        public static string GetConnectionString(string connectionName = "Database")
        {
            return ConfigurationManager.ConnectionStrings[connectionName].ConnectionString;
        }

        // takes a sql query as parameter and return a generic list
        // methode will be used in the Business Logic Layer
        public static List<T> LoadData<T>(string sql)
        {
            using (IDbConnection connection = new SqlConnection(GetConnectionString()))
            {
                return connection.Query<T>(sql).ToList();
            }
        }

        // takes a sql query & a generic data object as parameter and saves it in the db
        // methode will be used in the Business Logic Layer
        public static int SaveData<T>(string sql, T data)
        {
            using (IDbConnection connection = new SqlConnection(GetConnectionString()))
            {
                return connection.Execute(sql, data);
            }
        }
    }
}
```

***

## Business Logic

``` C#
namespace DataLibrary.BusinessLogic
{
    public static class CustomerProcessor
    {
        // gets all required data as parameters and converts them to a sql insert query
        // methode gets called by the controller of the MVC Project
        public static int CreateCustomer(string salutation, string surname, string forname,
            string street, string zip, string region, string country, string mail, DateTime birthday, string title = "", string phone = "", string smartphone = "")
        {
            Customer customer = new Customer
            {
                Salutation = salutation,
                Title = title,
                Surname = surname,
                Forname = forname,
                Street = street,                    
                Zip = zip,
                Region = region,
                Country = country,
                Mail = mail,
                Phone = phone,
                Smartphone = smartphone,
                Birthday = birthday,                 
            };

            string sql = @"insert into dbo.Customer (Salutation, Title, Surname, Forname, Street, Zip, Region, Country, Mail, Phone, Smartphone, Birthday)
                            values (@Salutation, @Title, @Surname, @Forname, @Street, @Zip, @Region, @Country, @Mail, @Phone, @Smartphone, @Birthday);";

            return SqlDataAccess.SaveData(sql, customer);
        }

        // method returns a list of all customers
        // gets called by a controller of the MVC Project
        public static List<Customer> LoadCustomers()
        {
            string sql = "select * from dbo.Customer;";

            return SqlDataAccess.LoadData<Customer>(sql);
        }
    }
}
```

***

## Models

I created a Frontend Customer Model to have a clear seperation between Front- & Backend.  Furthermore I used DataAnnotations & CustomeValidations to check the model for valid inputs.
The CustomeValidations can be found unter MVCApp/CustomeValidation. They are self-explanatory.

``` C#
namespace MVCApp.Models
{
    public class Customer
    {
        [Required]
        public string Salutation { get; set; }

        public string Title { get; set; }

        [Required]
        public string Surname { get; set; }

        [Required]
        public string Forname { get; set; }

        [Required]
        public string Street { get; set; }

        [Required]
        [DataType(DataType.PostalCode)]
        public string Zip { get; set; }

        [Required]
        public string Region { get; set; }
   
        [Required]
        [CountryValidation(ErrorMessage = "Enter a valide country")]
        public string Country { get; set; }

        [Required]
        // [DataType(DataType.EmailAddress)] isn't a good validation
        [MailValidation(ErrorMessage = "Enter a valide e-mail")]
        public string Mail { get; set; }

        [DataType(DataType.PhoneNumber)]
        public string Phone { get; set; }

        [DataType(DataType.PhoneNumber)]
        public string Smartphone { get; set; }

        [Required]
        [DataType(DataType.Date)]
        [AdultValidation(ErrorMessage = "Min. age is 18")]
        public DateTime Birthday { get; set; }

        [Required]
        [CheckboxValidation(ErrorMessage = "You must agree to our terms & conditions")]
        public bool CheckBoxTermsAndConditions { get; set; }

        [Display(Name = "Signature")]
        // required attribute is unnecessary because an empty signature pad 
        // has still a dataUri, so SignatureBase64String has always a value
        [SignatureValidation(ErrorMessage = "Please sign the form")]
        public string SignatureBase64String { get; set; }
    }
}
```

***

## Controller

<ins>3. Action Methodes</ins>
1. (HttpGet): Display Singup Form
2. (HttpGet): Display all signed up Customers
3. (HttpPost): Insert Customer

``` C#
namespace MVCApp.Controllers
{
    public class CustomerController : Controller
    {
        // shows the default page -> customer singup form
        public ActionResult SignUp()
        {
            // get all countries & iso code for the view
            List<string> countries = Country.GetAllCountries();
            if (countries.Count > 0)
            {
                this.Session["Countries"] = countries;
                ViewBag.CountryList = countries;
            }

            return View();
        }

        // displays all customers that are signed up
        public ActionResult ViewCustomers()
        {
            var data = CustomerProcessor.LoadCustomers();
            List<Customer> customers = new List<Customer>();

            foreach (var row in data)
            {
                customers.Add(new Customer
                {
                    Salutation = row.Salutation,
                    Title = row.Title,
                    Surname = row.Surname,
                    Forname = row.Forname,
                    Street = row.Street,
                    Zip = row.Zip,
                    Region = row.Region,
                    Country = row.Country,
                    Mail = row.Mail,
                    Phone = row.Phone,
                    Smartphone = row.Smartphone,
                    Birthday = row.Birthday,
                });
            }
            
            return View(customers);
        }

        // saves customer to database if the model is valid
        [HttpPost]
        [ValidateAntiForgeryToken]
        public ActionResult SignUp(Customer model)
        {
            // refresh viewbag data
            ViewBag.CountryList = this.Session["Countries"];

            // SignUp.cshtml validation is client sided & can be bypassed
            // this if-statement also validates model on the server side
            if (ModelState.IsValid)
            {
                int recordsCreated = CustomerProcessor.CreateCustomer(model.Salutation, model.Surname, model.Forname, model.Street, model.Zip, model.Region, model.Country, model.Mail, model.Birthday, model.Title, model.Phone, model.Smartphone);

                return View("Success");
            }

            return View();
        }
    }
}
```

***

## View

<ins>2. Views</ins>
1. Customer Signup Form
	1. includes JS for the Signature Pad
2. View all signed up Customers

Because the views are a little bit longer I don't show them here. Feel free to look at them in the projects repository.

***

## Signature Pad

I downloaded the JS file from the [repo](https://github.com/szimek/signature_pad) and put it into the /Scripts folder. Then I integrated it in my Views/Customer/SignUp.cshtml. To work properly I integrated it into /App_Start/BundleConfig.cs.

``` C#
namespace MVCApp
{
    public class BundleConfig
    {
        public static void RegisterBundles(BundleCollection bundles)
        {
            bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                        "~/Scripts/jquery-{version}.js"));

            bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(
                        "~/Scripts/jquery.validate*"));

            bundles.Add(new ScriptBundle("~/bundles/modernizr").Include(
                        "~/Scripts/modernizr-*"));

            bundles.Add(new ScriptBundle("~/bundles/bootstrap").Include(
                      "~/Scripts/bootstrap.js"));

            // integrated Signature Pad
            bundles.Add(new StyleBundle("~/bundles/signature_pad").Include(
                     "~/Scripts/signature_pad.umd.js"));

            bundles.Add(new StyleBundle("~/Content/css").Include(
                      "~/Content/bootstrap.css",
                      "~/Content/site.css"));
        }
    }
}
```

***

## Route Config

I changed the default route (App_Start/RouteConfig.cs) of the project to my CustomerController SignUp Methode.

``` C#
namespace MVCApp
{
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

            routes.MapRoute(
                name: "Default",
                url: "{controller}/{action}/{id}",
                // changed the default route of the project
                defaults: new { controller = "Customer", action = "SignUp", id = UrlParameter.Optional }
            );
        }
    }
}

```

***

## Tags

#Programming #Project #WEB #CSharp #Database #SQL