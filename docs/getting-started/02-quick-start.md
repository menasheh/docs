---
id: quick-start
title: Tutorial
---

In this tutorial, you will use Bit to build an app, bottom-up, using independent components. These components will be managed by a Bit workspace and shared to a remote scope on Bit.dev. You will then import and integrate a component from a remote scope into your own project.

## Install Bit

```sh
$ npm install @teambit/bit --global
```

## Initialize a new Bit workspace

Create a new Bit workspace by using the `bit init` command:

```sh
$ mkdir bad-jokes
$ cd bad-jokes
$ bit init --harmony
```

This will initialize a Bit workspace in our new `bad-jokes` directory. This commands creates two new assets for your project:

- `workspace.jsonc` the workspace' main configuration file..
- `components` is the root directory for all your components..

### Workspace configurations

To set our workspace configurations, we'll open the `workspace.jsonc` file. All root-level entries are different [aspects](TODO) (extensions) of Bit which you can configure to fit your setup.  
Configure the following (you can simply copy>paste this example):

```json
{
  /** 
  * the 'teambit.bit/worksapce' aspect control the workspace defaults and its metadata.
  */
  "teambit.bit/workspace": {
    /**
    * workspcae name
    */
    "name": "bad-jokes",
    /**
    * project logo. optional.
    */
    "icon": "https://image.flaticon.com/icons/svg/185/185034.svg",
    /**
    * The default scope for the components in the workspace - <company-name>.<scope>
    */
    "defaultScope": "teambit.bad-jokes",
  },
  /**
  * main configuration for bit's component dependency resolution.
  **/
  "teambit.bit/dependency-resolver": {
    /**
    * package manger for Bit to use
    */
    "packageManager": "teambit.bit/pnpm",
    /**
     * dependency policy allows to configure dependencies quickly and efficiently.
     * it helps to automate and manage dependency configuration for components.
    **/
    "policy": {
      /**
       * Dependency type is defined by the file that import it.
       * For example, library will be devDependency when the file that imports it is a test file.
      **/
      "dependencies": {
      },
    }
  },
  /**
   * workspace variants allow to set different subsets of configuration for components in your workspace.
   * this is extremely useful for upgrading, aligning and building components with a
   * new set of dependencies.
  **/
  "teambit.bit/variants": {
    /**
    * configure all components under directory `components/react`
    **/
    "components/react": {
      /**
      * configure the react environment on all components under the react directory.
      */
      "teambit.bit/react": {}
    },
    /**
    * configure all components under directory `components/modules`
    **/
    "components/modules": {
      /**
      * configure the react environment for all components in the `components/modules` directory.
      **/
      "teambit.bit/node": {},
      /**
       * these components has different scope
       **/
      "defaultScope": "teambit.toolbox"
    }
  }
}
```

## Start dev server

The [workspace UI](/docs/workspace/workspace-ui) displays all components in the workspace. It is a development tool that enables you to examine your components in different contexts and variations, and browse through their documentation. Start dev server and workspace UI:

```sh
$ bit start
```

## Add a new UI component

A [component](/docs/component/overview) is more than its set of implementation files. It has everything it needs to be used as an independent piece of software. Each component handled by a Bit workspace, must have all its files under the same directory:

```sh
$ mkdir components/ui/button
```

Now create the following files:

```sh
$ touch components/ui/button/index.ts
$ touch components/ui/button/button.tsx
$ touch components/ui/button/button.module.scss
```

And implement the component as follows:

<!--DOCUSAURUS_CODE_TABS-->
<!--index.ts-->

```javascript
export * from './button';
```

<!--button.tsx-->

```javascript
import React, { ButtonHTMLAttributes } from 'react';
import styles from './button.module.scss';

export interface IButton extends ButtonHTMLAttributes<HTMLButtonElement> {
  /** Choose between primary and secondary styling. */
  variant?: "primary" | "secondary";
}

export const Button = ({ children, variant = "primary", ...rest }: IButton) => {
  return (
    <button className={styles[variant]} {...rest}>
      {children}
    </button>
  );
};
```

<!--button.module.scss-->

```scss
@import url('https://fonts.googleapis.com/css2?family=Ubuntu:wght@700&display=swap');

.base {
  border-radius: 255px 15px 225px 15px/15px 225px 15px 255px;
  padding: 10px;
  color: #fff;
  font-size: 16px;
  font-family: 'Ubuntu', sans-serif;
  font-weight: 700;
  text-transform: uppercase;
  outline: none;
  transition: background-color 0.5s, border 0.5s;
    &:active {
      border-color: #313131;
      background-color: #fff;
      color: #000;
    }
    &:disabled{
      border-color: #a5a5a5;
      background-color: #a5a5a5;
      &:hover {
        border-color: #a5a5a5;
        background-color: #a5a5a5;
      }
  }
}
.primary {
    border: solid 7px #ab3636;
    background-color: #ab3636;
      &:hover {
        border-color: #8b1b1b;
        background-color: #8b1b1b;
      }
  }
  .secondary {
    border: solid 7px #363eab;
    background-color: #363eab;
      &:hover {
        border-color: #434cc5;
        background-color: #434cc5;
      }
  }
```
<!--END_DOCUSAURUS_CODE_TABS-->

### Install dependencies

The above button component uses the `classnames` library. To install it, use `bit install`:

```bash
$ bit install classnames
```

Bit will make sure to register this package in the workspace configurations file (`workspace.jsonc`), under the `dependency-resolver` entry (a Bit workspace has no `package.json` file)

```json
  "@teambit.bit/dependency-resolver": {
    "packageManager": "@teambit.bit/pnpm",
    "strictPeerDependencies": true,
    "policy": {
      "dependencies": {
        "classnames": "^2.2.6"
      },
```

### Track the Component

To track a component by Bit, use:

```sh
$ bit add components/ui/button
```

Now browse to the local development server at [http://localhost:3000]() to see that the component is managed by Bit.

### Review the documentation

Bit automates component documentation by parsing your code and creating docs using a pre-defined template. When you track a component, Bit automatically adds it to the workspace UI application. The component's main overview page contains its documentation.

### Add compositions

Use Compositions to render the component in isolation and test is in various cases. Create the `compositions` file as part of the component:

```sh
$ touch components/ui/button/button.composition.tsx
```

Add the following lines to `components/ui/button/button.composition.tsx`

```tsx
import React from "react";
import { Button } from "./button";

export const PrimaryButton = () => {
  return (
    <Button variant="primary" onClick={() => alert("Clicked!")}>
      Primary Button
    </Button>
  );
};

export const SecondaryButton = () => {
  return <Button variant="secondary">Secondary Button</Button>;
};

export const DisabledButton = () => {
  return (
    <Button disabled variant="primary">
      Disabled Button
    </Button>
  );
};
```

Head over to the [compositions tab](https://localhost:3000/base/button/~compositions) to see all component compositions.

1. Each named export is a composition.
1. Composition name is a prettified version of the export name.

### Add tests

As you recall, our Bit workspace uses the `@teambit/react` environment extension. This environment is pre-configured to use Jest as a test runner (which means we don't need to worry about setting it up).  
To simplify our UI testing, we'll also make use of the following libraries:

```bash
$ bit install @testing-library/react @testing-library/jest-dom
```

Notice how we didn't set these packages as Dev Dependencies. Bit determines that by analyzing the way they are used in this workspace.  
Let's start by creating our test file (in the button component directory)

```bash
$ touch button.spec.js
```

And place a simple test to validate that it renders:

```jsx
import "@testing-library/jest-dom/extend-expect";
import React from "react";
import { render } from "@testing-library/react";
import { Button } from "./button";

test("button renders", () => {
  const { getByText } = render(<Button>test button</Button>);
  const testButton = getByText(/test button/i);
  expect(testButton).toBeInTheDocument();
});
```

Our tests will run whenever we [tag](/docs/versioning/tracking-changes) the relevant component. If a test fails, the tagging action will be aborted.  
We can also run the tests manually:

```bash
$ bit test
```

### Use documentation template

The `abstract` and `tags` define the component description and related categories. Both are generated automatically by Bit. To override Bit's auto-generated data, use the `abstract` and `tags` variables, in the component's `*.doc.tsx` file.  
For example:

```js
export const abstract = "An imperfect button";

export const tags = ["react", "typescript", "button"];
```

#### Extend the documentation with a custom component

The custom JSX slot gives you the freedom to extend the documentation page as you like. To use it, create a regular React component in the `doc` file, and export it as `default`.  
For example, let's create a 'Guidelines' section for a `Button` component documentation:

```tsx
export default function() {
  const wrapper = {
    border: "1px solid #e0ddd8",
    borderRadius: "5px",
    padding: "25px",
    marginBottom: "25px"
  };
  return (
    <div style={wrapper}>
      <p style={{ fontWeight: 700 }}>Guidelines</p>
      <br />
      <ul style={{ listStyleType: "circle", paddingLeft: "25px" }}>
        <li>
          Place buttons where users expect to find them. Do not force users to
          "hunt for buttons".
        </li>
        <li>
          Do not use generic labels for your buttons. Use verbs that clearly
          explain the button's function.
        </li>
        <li>Size buttons in proportion to their importance</li>
      </ul>
    </div>
  );
}
```

## Add a new React hook

Add a react hook to handle the data fetching for bad-jokes.  
In Bit, even hooks managed as components. So we start by creating the component directory and files:

```sh
$ mkdir -p hooks/use-get-joke
$ touch hooks/use-get-joke/index.ts
$ touch hooks/use-get-joke/use-get-joke.ts
```

And implement as follows:

<!--DOCUSAURUS_CODE_TABS-->

<!--index.ts-->

```javascript
export * from './use-get-jokes';
```

<!--use-get-jokes.ts-->

```ts
import { useState, useEffect } from "react";

export const useGetJokes = (): [
  () => Promise<void>,
  string[],
  boolean,
  string
] => {
  const [joke, setJoke] = useState([""]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState("");
  const endpoint =
    "https://sv443.net/jokeapi/v2/joke/Any?blacklistFlags=racist&type=single";

  const getJoke = async () => {
    setIsLoading(true);
    try {
      const res = await fetch(endpoint);
      const data = await res.json();
      const dataArr = data.joke.split("\n");
      setJoke(dataArr);
      if (error) setError("");
      setIsLoading(false);
    } catch (err) {
      setError(err.message);
      setIsLoading(false);
    }
  };
  useEffect(() => {
    getJoke();
  }, []);

  return [getJoke, joke, isLoading, error];
};
```

<!--END_DOCUSAURUS_CODE_TABS-->

### Track hook component

```sh
$ bit add hooks/use-get-joke --namespace hooks
```

> **Review the documentation**
>
> As with any other component, Bit automatically documents your hook. Head over to the component page on the workspace UI to review it.

### Add live examples

Examples are descriptions and playable code that instruct on how to use a component. Examples are set using the `examples` variable in the `<component>.docs.tsx` file.  
The `examples` variable receives an array of objects, each representing a single example and each contains the following data (keys):

- `scope` - An `object` with all relevant imports.
- `title` - A `string` for the example title.
- `description` - A `string` for the example description.
- `code` - A `string` (template literal) for the example code.

Let's create an example for the `Card` component:


```sh
$ touch ./path/to/component/folder/card.docs.tsx
```

Inside that file, we'll import `Card` and set the `examples` variable with a single object.

```tsx
import {useGetJokes} from './use-get-jokes'

export const examples = [
  {
    scope: {
      useGetJokes
    },
    title: "Using useGetJokes",
    description:
      "This hook uses a jokes api. It returns the data-fetching function (getJoke), the data (joke), the state of the 'getJoke' function and an error message (or an empty string).",
    code: `() => {
        const [getJoke, joke, isLoading, error] = useGetJokes();
        return (
            <div>
                <div>
                    {error || joke.map(line => <p>{line}</p>)}
                </div>
                <div>
                    <button disabled={isLoading} onClick={getJoke}>
                        {isLoading ? 'loading...' : "fetch a joke"}
                    </button>
                </div>
            </div>
        )
    }`
  }
];
```

## Compose an app

This will be our (app) component structure:

```sh
├── components
  ├── apps
    ├── jokes-viewer
       ├── jokes-viewer.tsx
       ├── jokes-viewer.module.scss
       ├── jokes-viewer.spec.jsx
       └──  index.ts
```

### Import tracked components

Bit links tracked components to the`node_modules` directory. We'll use these links when referencing components. For example, in our `jokes-viewer.tsx` file we'll import the "button" UI component and "getJokes" hook, like so:

```tsx
import {Button} from '@teambit/bad-jokes.ui.button';
import {useGetJokes} from '@teambit/bad-jokes.hooks.use-get-jokes'
```

We'll use them like so:

```tsx
/** Retrieves and displays bad jokes */
export const BadJokesViewer = () => {

    const [getJoke, joke, isLoading, error] = useGetJokes();

    return (
        <div className={styles.badJokesViewer}>
            <div className={styles.contentWrapper}>
                {error || joke.map(line => <p>{line}</p>)}
            </div>
            <div>
                <Button disabled={isLoading} onClick={getJoke}>
                    {isLoading ? 'loading...' : "another one, please"}
                </Button>
            </div>
        </div>
    )
}
```

All that's left is to style the component. In the `jokes-viewer.module.scss`:

```scss
@import url('https://fonts.googleapis.com/css2?family=Indie+Flower&display=swap');

.badJokesViewer {
    border-radius: 255px 15px 225px 15px/15px 225px 15px 255px;
    border: solid 7px #41403E;
    padding: 25px;
    box-shadow: 20px 38px 34px -26px hsla(0,0%,0%,.2);
    font-family: 'Indie Flower', cursive;
    font-size: 20px;
    max-width: 400px;
    transition: box-shadow 0.5s;
        &:hover {
            box-shadow:2px 8px 4px -6px hsla(0,0%,0%,.3);
          }        
  }

.contentWrapper{
    margin-bottom: 50px;
}
```

### Add a composition

Our app component, "jokes-viewer", is structured like so:

## Version components

## Push to github

### Set up a bit.dev account

### Create remote scope

### Configure scope in workspace

### Connect bit.dev account to the github repo

## Reuse components in another project

## vendor component