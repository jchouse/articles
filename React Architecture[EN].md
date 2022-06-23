# React App File Structure.

## Intro
The problem of organizing files within an application is starting when more than one person works on the project and the size of the code base growss to such level, that it's difficult to keep everything in mind. That's why we are always trying to make file structure more obvious, so it can be used comfortably and quickly.

React doesn't dictate it's application architecture, leaving complete freedom in how the application files are organized: https://reactjs.org/docs/faq-structure.html

I've spent quite a time to find approach for my project, but first, we need some context:
1. Main language - TypeScript
1. UI library - React
1. State manager - MobX
1. Authoring tool for CSS - JSS Styles
1. Testing library - JEST

Let's talk about the details. Nobody knows why, but it was decided to rewrite the 5 years old project from ES+Redux to TS+MobX, which already had the minimal set of features to be able to sell it and become profitable. Of course, if I was there from the start I would prefer TS+MobX to ES+Redux, but I would never agree to rewrite my existing project from one stack to another. Feel free to write in the comments if you want to hear this story.

Also, in the process of finding a file structure that will fit to project, I've read some articles:
1) https://www.taniarascia.com/react-architecture-directory-structure/
1) https://www.robinwieruch.de/react-folder-structure/

## Existing solutions
Our project was originally structured for a global state, but at the same time it had a pseudo-modular structure with redundant nesting. Why do I consider our previous structure pseudo-modular? The level of dependencies between the modules was very high and did not allow us to move one of the modules into a separate repository without an multiple code duplication, which meant that this did not give us anything but a more complex structure and extensive nesting. Roughly it looked like this:

```
.
├── /modules
│   └── /ModuleName1
│        └── /src
│            └── /js
│                ├── /api
│                ├── /actions
│                ├── /components
│                ├── /containers
│                ├── /reducers
│                └── /stores
└── /src
    ├── /api
    ├── /assets
    ├── /components
    ├── /constants
    ├── /containers
    ├── /icons
    ├── /reducers
    ├── /stores
    ├── /styles
    ├── /utils
    ├── index.js
    └── App.js
```

It is safe to assume that this solution is based on the acceptance of the frontend developer community, which is based on the functional naming of the code inside. Here is an example:

```
.
└── /src
    ├── /actions
    ├── /assets
    ├── /components
    ├── /constants
    ├── /containers
    ├── /icons
    ├── /reducers
    ├── /stores
    ├── /styles
    ├── /utils
    ├── index.js
    └── App.js
```
Both approaches have the right to exist, and to be fair, the second variant with help of aliases for WebPack and well organized index.ts, will close the problem of redundant nesting. However, it requires additional steps to maintain the code.

So, what has changed with the moving away from the Redux global store (and all related libraries Thunk, reselect, Recompose, etc.)? It became possible to write atomic storages, i.e. when the storages are written specifically for the component and can be connected at any level in the provider. Of course, this approach requires a different approach to write the component. There is no urgent need to link Reducers in Combine, and collect them throughout the project. Or maybe you don't need to collect them at all over components and put them in the same directory? I don't think it will greatly simplifies the perception, because you have to start to write the complex imports (aliases) in the second case, and the directory Reducers threatens to grow to 15-20 files at one level, which will make worse search and visual perception.

During my search for the optimal solution, I've found another option, proposed by Robin Wieruch (link at the beginning of the article):

```
- src/
--- App/
----- index.js
----- component.js
----- test.js
----- style.css
--- List/
----- index.js
----- component.js
----- test.js
----- style.css
----- ListItem/
------- index.js
------- component.js
------- test.js
------- style.css
```

There are some interesting thoughts in this article, that resonates with my thoughts about the structure of the file structure in React. But it has it's own flaws, that are obvious to me. The first one that catches my eye - quick access to files based on name which is simply missing, as most popular code editors and IDEs allows you to quickly find a file just by first letters in the name, with this approach you can't do that. Type component.js in the search for this layout and you'll see what I mean.

## What we came up with
As a result, after analyzing the existing solutions and our experience, we developed our approach to organizing the files within the project:

1. After all, all the code is in `src`. And `src` is the root directory for all the code. It is possible to fix this with an alias in webpack.

```
  .
  └── /src
```

There is no reason to put any pages or components into `modules`. Any code that should be put into a separate application module should be discussed separately with the help of a special decision framework, where the reasons, consequences and process will be described separately.

2. At the root of the repository, the main division is by the name of the component or page.

```
  .
  └── /src
      ├── /App
      │   └── App.ts
      ├── /Header
      │   └── Header.ts
      ├── /Portal
      │   └── Portal.ts
      └── /Creation
          └── Creation.ts
```

3. We abandon `components`, `containers`, `stores`, `api`, `styles` outside the context of a component. And inside of the file structure, there are no longer unnecessary nesting, but folders display the purpose and content of the folder.

```
  .
  └── /src
      ├── /App
      ├── /Header
      ├── /Portal
      │   ├── Portal.api.ts
      │   ├── Portal.store.ts
      │   ├── Portal.interface.ts
      │   ├── Portal.styles.ts
      │   └── Portal.ts
      └── /Creation
```

4. We decided to keep only `assets` and `tests` folders to distinguish files not directly related to development, but they should be placed as close as possible to the place of direct use. If desired, an underscore symbol may be used at the beginning and/or end of the name to ensure that such directories are placed at the beginning of the file list.

```
  .
  └── /src
      ├── /App
      ├── /Header
      ├── /Portal
      │   ├── /__tests__
      │   ├── /_assets
      │   ├── Portal.api.ts
      │   ├── Portal.store.ts
      │   ├── Portal.interface.ts
      │   ├── Portal.styles.ts
      │   └── Portal.ts
      └── /Creation
```

5. If it is necessary to reduce the length of the file to improve the readability of the code, parts of the logic can be put in separate files (limited by the allowable length of the file is best set at the linter level). Such parts should be placed with reasonable nesting. Only one level of nesting is allowed for each component. There is no sense to go one or two levels inside.

```
  .
  └── /src
      ├── /App
      ├── /Header
      ├── /Portal
      │   ├── /_tests
      │   ├── /_assets
      │   ├── /PortalSearchBar
      │   │   ├── PortalSearchBar.styles.ts
      │   │   └── PortalSearchBar.ts
      │   ├── Portal.api.ts
      │   ├── Portal.store.ts
      │   ├── Portal.interface.ts
      │   ├── Portal.styles.ts
      │   └── Portal.ts
      └── /Creation
```

## Conclusions.
In my opinion, the proposed structure:
1. helps the developer to keep in mind MVV/MVC paradigm
1. Plan component design in advance
1. Spend less time searching for the components, inside file tree and quick access panel as well

Is it 100% optimal and suitable model for everyone? Definitely not, but in our project, it obviously is.
So if you have thoughts or comments on this, please write to me [@jchouse](https://github.com/jchouse)

Thanks for:
- my codebuddy [@dmytro1ef2](https://github.com/dmytro1ef2)
- translation editing [@vadimfrolov](https://github.com/vadimfrolov)