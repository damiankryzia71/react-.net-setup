# Connect a React frontend to a ASP.NET MVC application

## Prerequisites
Install [Node.js](https://nodejs.org/en)

## 1. Create the React app
Open a new terminal and run this command.
```bash
npx create vite@latest App --template react-ts
```
Select React and Typescript if prompted.
This will create a new React + Typescript application with Vite as the build tool.

## 2. Install React Router for frontend routes
Run this command in the project directory.
```bash
npm install react-router
```

## 3. Set up frontend routes
Edit the `main.tsx` file to set up browser routes.

Read more about React Router [here](https://reactrouter.com/home).

In this example, the `/*` symbol makes it so that all paths that are not hard coded will default to the home page.
```ts
import { BrowserRouter, Routes, Route } from 'react-router'
import { createRoot } from 'react-dom/client'
import Home from './modules/home/Home.tsx'
import Weather from './modules/weather/Weather.tsx'

createRoot(document.getElementById('root')!).render(
  <BrowserRouter>
    <Routes>
      <Route path="/*" element={<Home />} />
      <Route path="weather" element={<Weather />} />
    </Routes>
  </BrowserRouter>,
)
```

## 4. Run the development server
Run this command in the project directory to start the development server.
```bash
npm run dev
```
NOTE: To make API calls to the ASP.NET MVC application during React development, appropriate CORS rules must be set in the ASP.NET application. This allows the `localhost` server running the React application to communicate with the `localhost` server running the ASP.NET MVC application.

Moreover, during development, API calls must be made to the full address, for example:
```ts
fetch("https://localhost:7288/api/example")
```
There may be ways to omit this but none worked during testing. Read more [here](https://vite.dev/config/server-options#server-proxy).

## 5. Build the React application
Once the React application is ready to be built, you can start integrating it with the ASP.NET MVC application.

First, change API call paths to relative paths. For example:
```ts
fetch("https://localhost:7288/api/example")
```
changes to:
```ts
fetch("/api/example")
```

When ready, run this command in the project directory.
```bash
npm run build
```
This created a `dist` folder that contains the web application's static files. Move this folder to your ASP.NET MVC application directory.

## 6. Integrate the React frontend with the ASP.NET MVC backend
There are a few things to set up in the ASP.NET MVC application.

The application will serve a single view (`Index`) which will serve as the entry point to the React frontend application.

First, set up your `RouteConfig.cs` to direct all routes to the `Index` view.
```cs
    public class RouteConfig
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.IgnoreRoute("{resource}.axd/{*pathInfo}");

            routes.MapRoute(
                name: "Index",
                url: "{*catch-all}",
                defaults: new { controller = "Home", action = "Index"}
            );
        }
    }
```
Then, include the `Index` view in your `HomeController.cs`
```cs
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }
    }
```
Finally, create the `Index.cshtml` view file insie `Views/Home/` and set up the content like this.
Make sure that the `script` tag points to the appropriate `.js` file in `dist`.
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>My React App</title>
    </head>
    <body>
        <div id="root"></div>
        <script src="~/dist/assets/index.js"></script>
    </body>
</html>
```

When building the ASP.NET application, all routes will redirect to the `Index` view, from where React Router will take over browser routing. If you have a web API controller set up, it should continue working properly without any changes.
