# React App File Structure.

## Intro 
The problem of organizing files within an application does not arise from nothing, as a rule, it is thought about when more than one person works on the project and the size of the code base grows to such a level that it is difficult to keep in memory which file is responsible for which part of the application. That is, we always try to make the structure so that it can be used comfortably and quickly.

React doesn't dictate its application architecture, leaving complete discretion in how the application files are organized: https://reactjs.org/docs/faq-structure.html

To give a little context, I will tell you about the solutions and technologies used within our project:
1. The main language of the TypeScript application.
1. The main library, as I mentioned: React.
1. State manager MobX.
1. JSS Styles
1. Testing - JEST

I will add a little more. Initially, all the code was written in ES+Redux. Then, but history is silent and I can only guess why, it was decided to rewrite the 5 years old project, which had almost the entire minimal set of features to be able to sell it and not to work at a loss, on TS+MobX. Of course, if I was given a choice now and asked what I want to use for writing a new project I would choose Redux on TS_MobX, but then again if I was offered to rewrite my existing project from one stack to another I would probably refuse. Feel free to write in the comments if you want to hear this story.

Also, in the process of finding a solution, I used the achievements and thoughts of the authors of the articles:
1) https://www.taniarascia.com/react-architecture-directory-structure/
1) https://www.robinwieruch.de/react-folder-structure/

## Existing solutions
Our project was originally structured for a global state, but at the same time had a pseudo-modular structure with redundant nesting. Why do I consider our previous structure pseudo-modular? The level of dependencies between the modules was very high and did not allow us to move one of the modules into a separate repository without an infernal duplication, which meant that this did not give us anything but a more complex structure and extensive nesting. Roughly it looks like this:

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

It is safe to assume that this solution is based on the acceptance in the community of frontend developers' division of files, which is based on the functional naming of the code inside, here is an example of such a variant.

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
Both approaches have the right to exist and, to be fair, the example above, if you use aliases for WebPack and index.ts skillfully, will negate the problem of redundant nesting. However, they require additional steps from the owners of the repository code and a more thorough code review.

What has changed with the rejection of the Redux global store (and all related libraries Thunk, reselect, Recompose, etc.). It became possible to write atomic storages, i.e. when the storages are written specifically for the component and can be connected at any level in the provider. Of course, this approach requires a different approach to writing the component. There is no urgent need to link Reducers in Combine. And go around collecting them throughout the project. Or maybe you don't collect them all over components and put them in the same directory? I don't think it greatly simplifies the perception, in the first case, you again start complex imports (aliases) in the second case, the directory Reducers threatens to grow to 15-20 files at one level, which will not help either search or visual perception.

During my search for the optimal solution, I came across another option that looks like this, proposed by Robin Wieruch (link at the beginning of the article)

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

There are some interesting thoughts in this that resonate with my thoughts about the structure of the project files in React. But there are also shortcomings that are obvious to me. The first one that catches my eye quick access to files based on name is simply missing, as most popular code editors and IDEs allow you to quickly open a file just by starting to type in its name here you obviously can't do that. Type component.js in the search for this layout and you'll see what I mean.

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

3. We abandon `components`, `containers`, `stores`, `api`, `styles` outside the context of a component. And inside the component, they are no longer organizational and create unnecessary nesting, but should display the purpose and content of the file.

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

4. We decided to keep only `assets` and `tests` to distinguish files not directly related to development, but they should be placed as close as possible to the place of direct use. If desired, an underscore symbol may be used at the beginning and/or end of the name to ensure that such directories are placed at the beginning of the file list.

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

5. If it is necessary to reduce the length of the file to improve the readability of the code, parts of the logic can be put in separate files (the allowable length of the file is best set at the linter level). Such parts should be placed respecting nesting. Only one level of nesting is allowed for each component. It is not necessary to go one or two levels inside.

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
In my opinion. The proposed structure helps the developer to think in the MVV/MVC paradigm. Plan component design in advance. Spend less time searching for components, both just in the tree and through the quick access panel.

Is the proposed model 100% optimal and suitable for everyone? I am far from this thought. But in our project, it obviously is. So if you have thoughts or comments on this, please write to me.