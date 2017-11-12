UI framework written in rust. 
apps are programmed in html, css, and rust or something that can have rust bindings. 
build time features/macros should be available to make binding errors between rust/html/css/resources to be build time errors instead of runtime errors. 
should run on any platform, android, ios, desktop, web, 
should port cleanly to web platform. when built for web platform it should use the underlying browsers html/css/dom implementation, instead of bringing its own, allowing for smaller downloads. 
It should allow the app maximum control for the app, dictating as little as possible. 
It should provide defaults for most anything. 
It should have a rustic api that nicely deals with cross thread concerns.
It should not require the users to know the details of the GC implementation.
It should only do UI and platform abstraction. It should not be an overall app framework. 
It should allow the users to create and share their own components. 
     these components should allow for static dispatch or dynamic dispatch as the user wishes. 
It should include some sort of extension to html references/links to allow components to refernce compiled in resources and executable code. This extension should also allow components to reference other components. possibly from within html?
Components should be composable?

react or other frameworks could be ran on top of this. 

should use gpu if available, or cpu rendering if not. Should make choice automatically. 

goals: 
    make a nice ui framework in rust.
    free the web from javascript
    make a tool good for teaching in gradeSchoolCS. sqlplayground should be written using this. as they learn rust and html it can serve as an example software. 
