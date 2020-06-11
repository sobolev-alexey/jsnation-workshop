# Build a React-based Theme for WordPress with Frontity

This repo contains the complete theme created for 'Build a React-based Theme for WordPress with Frontity', a workshop/webinar held on 15 June 2020 at the [JS Nation Live conference](https://live.jsnation.com/).

This readme also contains full instructions for creating the project from scratch.

## Table of Contents

1. [Create a Frontity Project](#create-a-frontity-project)
2. [Create a custom theme from scratch](#create-a-custom-theme-from-scratch)
3. [Modify the first component](#modify-the-first-component)
4. [Connect it to the state](#connect-it-to-the-state)
5. [Add a menu](#add-a-menu)
6. [Use data from the current URL](#use-data-from-the-current-URL)
7. [Display the list of posts](#display-the-list-of-posts)
8. [Display the content of posts](#display-the-content-of-posts)
9. [Display the content of posts and pages separately](#display-the-content-of-posts-and-pages-separately)
10. [Add some style](#add-some-style)
11. Use state and actions
12. Add tags to the <head>

## 1. Create a Frontity Project

The first thing we are going to do is create a new Frontity project.

To do so, open up your terminal, navigate to the folder where you want to install your new project, and run this command:

```bash
npx frontity create jsnation-frontity
```

When the install process finishes, you will have a new sub-folder called `jsnation-frontity` containing your project’s code.

Start a development server to check that everything is working:

```bash
cd jsnation-frontity
npx frontity dev
```

> `npx` downloads an npm package to run just this one time and then removes it from your computer. [learn more](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)

Now open `http://localhost:3000` in your browser (if it didn't open automatically) to see your first Frontity project. At the moment it's using the "starter theme" that comes with Frontity, i.e. @frontity/mars-theme, and it is connected to a test WordPress site (https://test.frontity.io).

The next step is to change the project’s settings to point to the REST API of our own website (https://app-5ec4037dc1ac18016c052959.closte.com) - yes, the URL is pretty ugly, but it's just for test purposes. 😀

- Open the file `frontity.settings.js` file. This file contains the configuration for the Frontity packages that you are using in your project.

- Change the value of the `api` field of the `@frontity/wp-source package`.

Replace this:
```js
state: {
  source: {
    api: "https://test.frontity.io/wp-json"
  }
}
```
With this:
```js
state: {
  source: {
    api: "https://app-5ec4037dc1ac18016c052959.closte.com/wp-json"
  }
}
```
Refresh the page browser to see all the posts on the website. What you can see now is content generated by the [WordPress Theme Unit Test](https://github.com/WPTT/theme-unit-test). This provides a set of typical content for a WordPress site and includes a number of edge cases, such as very long titles. This content set is useful for designing, creating and testing traditional PHP based WordPress themes, but we will find it useful for our purposes too.

## 2. Create a custom theme from scratch

Instead of using the default starter theme (@frontity/mars-theme) we are going to create a new package for our theme's code.

To do so, stop the previous process (CONTROL + C), and then run this command in your terminal:

```bash
npx frontity create-package jsnation-theme
```

You will be asked what namespace to use. Since you are creating a theme, you can use `theme`.

When the process is complete you will have a new folder called `/packages/jsnation-theme`. This is where we will be doing most of our work to build the theme.

The first thing to do is to remove `@frontity/mars-theme` from your settings and replace it with `jsnation-theme`.

Remove the following from the file `frontity.settings.js`:

```js
{
  name: "@frontity/mars-theme",
  state: {
    theme: {
      menu: [
        ["Home", "/"],
        ["Nature", "/category/nature/"],
        ["Travel", "/category/travel/"],
        ["Japan", "/tag/japan/"],
        ["About Us", "/about-us/"]
      ],
      featured: {
        showOnList: false,
        showOnPost: false
      }
    }
  }
},
```

And replace it with:

```js
{
  "name": "jsnation-theme"
},
```

And then, run this command again:

```bash
npx frontity dev
```

## 3. Modify the first component

Let's start by modifying the `<Root>` component in the `/packages/jsnation-theme/src/index.js` file so that it returns a `<h1>` containing the text “Frontity Workshop”.

```jsx
// File: /packages/jsnation-theme/src/index.js

const Root = () => {
  return (
    <>
      <h1>Frontity Workshop</h1>
    </>
  );
};
```

The content in the browser should automatically update as we have changed a file within the `/packages` folder.

Now, let's move the <Root> component into its own file.

Create a new folder called `theme-files` inside `/packages/jsnation-theme/src`. This is where we will create all the component files for our theme. Then, inside `/packages/jsnation-theme/src/theme-files` create a new file called `index.js`.

Add the following code to the new file *(which we will henceforth refer to as the 'root component' file)*.

```jsx
// File: /packages/jsnation-theme/src/theme-files/index.js

import React from "react";

const Root = () => {
  return (
    <>
      <h1>Frontity Workshop</h1>
    </>
  );
};

export default Root;
```

We now need to import it into the file `/packages/jsnation-theme/src/index.js` which you can modify as follows:

```jsx
// File: /packages/jsnation-theme/src/index.js

import Root from "./theme-files";

const jsNationTheme = {
  name: "jsnation-theme",
  roots: {
    theme: Root
  },
  state: {
      theme: {}
  },
  actions: {
      theme: {}
  }
};

export default jsNationTheme
```

Save both files and check that everything is still working in the browser.

## 4. Connect it to the state

Let’s connect the `<Root>` component to the Frontity state using `connect`. This allows us to access data stored in the state.

We need to `import {connect} from "frontity"`, pass `state` as a prop to our component, and finally export the connected component.

Then we add a `<p>` element to our component to display the URL we are currently in using `state.router.link`.

```jsx
// File: /packages/jsnation-theme/src/theme-files/index.js

import React from "react";
import { connect } from "frontity";

const Root = ({ state }) => {
  return (
    <>
      <h1>Frontity Workshop</h1>
      <p>Current URL: {state.router.link}</p>
    </>
  );
};

export default connect(Root);
```
You can try changing the URL to something like http://localhost:3000/hello-frontity to see how the `state.router.link` changes.

## 5. Add a menu

Now let's create a `<Link>` component in a new file `link.js`:

```jsx
// File: /packages/jsnation-theme/src/theme-files/link.js

import React from "react";
import { connect } from "frontity";

const Link = ({ href, actions, children }) => {
  return (
    <div>
      <a
        href={href}
        onClick={ e => {
          e.preventDefault();
          actions.router.set(href);
        }}
      >
        {children}
      </a>
    </div>
  );
};

export default connect(Link);
```

We now have to import the `Link` component into our `Root` component. Then we add a menu with three `Link` items.


```jsx
// File: /packages/jsnation-theme/src/theme-files/index.js

import Link from "./Link";

const Root = ({ state }) => {
  return (
    <>
      <h1>Frontity Workshop</h1>
      <p>Current URL: {state.router.link}</p>
      <nav>
        <Link href="/">Home</Link>
        <Link href="/page/2">More posts</Link>
        <Link href="/lorem-ipsum">Lorem Ipsum</Link>
      </nav>
    </>
  );
};
```

## 6. Use data from the current URL

In order to gain a better understanding of Frontity, let’s dig a little deeper and investigate how it works below the surface.

To do so, access `http://localhost:3000/lorem-ipsum/` in the browser and open the console. In the console type `frontity.state` to see the Frontity state. This is the same state that the components and actions have access to.

<p>
  <img alt="Frontity in the console" src="assets/console-1.png">
</p>

> Frontity uses [ES2015 Proxies](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy), so you have to open the property [[Target]] in order to see the state.

You will see Frontity's global state, including the general properties of your Frontity project. You can also see information about the `router`, including the `state.router.link` that we used earlier, and `source`, the package that connects Frontity to your WordPress site.

Let’s take a look at `state.source.data`. This is where the information for each URL is stored. If you inspect `/lorem-ipsum/`, you can see that it’s a page, and that it has the ID 146.

<p>
  <img alt="Frontity in the console" src="assets/console-2.png" width="600">
</p>

With that information, we can access the data (title, content, etc) of that page with `state.source.page[146]`:

<p>
  <img alt="Frontity in the console" src="assets/console-3.png">
</p>

As you navigate from one URL to another, the package `@frontity/wp-source` automatically fetches everything you need and stores it in `state.source`.

If we open the Network tab (in the browser's devtools) and click on the menu to go to Home (Homepage), you will see that a call to the REST API is made to get the latest posts.

Take another look at `state.source.data`. You will notice that it's been populated with much more data than before.

Please note that instead of using `state.source.data[url]` it’s better to use `state.source.get(url)`. This ensures that URLs always include the final slash (/).

So now let’s inspect the homepage using state.source.get("/"):

<p>
  <img alt="Frontity in the console" src="assets/console-4.png" width="700">
</p>

As you can see, it has several interesting properties such as `isHome`, `isArchive`, and an array of `items`. If the homepage were a category it would have an `isCategory` property. If it were a post it would have a `isPost` property, etc...

To wrap up this section let's use all of this in our code.

For our next step we obtain the information about the current link (`state.router.link`) and use it to see if it’s a `list`, a `post`, or a `page`.

```jsx
// File: /packages/jsnation-theme/src/theme-files/index.js

const Root = ({ state }) => {
  const data = state.source.get(state.router.link);

  return (
    <>
      <h1>Frontity Workshop</h1>
      <p>Current URL: {state.router.link}</p>
      <nav>
        <Link href="/">Home</Link>
        <Link href="/page/2">More posts</Link>
        <Link href="/lorem-ipsum">Lorem Ipsum</Link>
      </nav>
      <hr />
      <main>
        {data.isArchive && <div>This is a list</div>}
        {data.isPost && <div>This is a post</div>}
        {data.isPage && <div>This is a page</div>}
      </main>
    </>
  );
};
```

## 7. Display the list of posts

In order to display the list of posts, create a component called `<List>` in a new file `list.js`.  This will show the information in `state.source.data`, namely the `type`, `id` and `link` of each post.

```jsx
// File: /packages/jsnation-theme/src/theme-files/list.js

import React from "react";
import { connect } from "frontity";

const List = ({ state }) => {
  const data = state.source.get(state.router.link);

  return (
    <div>
      {data.items.map(item => {
        return (
          <div key={item.id}>
            {item.type} – {item.id} – {item.link}
          </div>
        );
      })}
    </div>
  );
};

export default connect(List);
```

We need to import it into our root component and use it.

```jsx
// File:  /packages/jsnation-theme/src/theme-files/index.js

// ...
import List from "./List";

const Root = ({ state }) => {
  const data = state.source.get(state.router.link);

  return (
    <>
      {/* ... */}
      <main>
        {data.isArchive && <List />}
        {data.isPost && <div>This is a post</div>}
        {data.isPage && <div>This is a page</div>}
      </main>
    </>
  );
};
```

And now let's change the `<List>` component to use the information about each of the posts to show the title and turn it into a link.

```jsx
// File: /packages/jsnation-theme/src/theme-files/list.js

import  React  from  " react " ;
import { connect } from  " frontity " ;
import  Link  from  " ./Link " ;

const  List  = ({ state }) => {
   const  data  =  state.source.get(state.router.link );

  return (
    < div >
       { data.items.map( item => {
         const post = state.source.post[item.id]
         return (
           <Link key={item.id} href={post.link}>
              {post.title.rendered}
           </Link>
         );
      }) }
    </ div >
  );
};
```

## 8. Display the content of posts

Create a new file called `post.js`. This will contain the `<Post>` component which we will use to display the title and the content of the posts.

```jsx
// File: /packages/jsnation-theme/src/theme-files/post.js

import React from "react";
import { connect } from "frontity";

const Post = ({ state }) => {
  const data = state.source.get(state.router.link);
  const post = state.source[data.type][data.id];

  return (
    <div>
      <h2>{post.title.rendered}</h2>
      <div dangerouslySetInnerHTML={{ __html: post.content.rendered }} />
    </div>
  );
};

export default connect(Post);
```

And, as before, import it into the root component file and use it to display posts and pages.

```jsx
// File: /packages/jsnation-theme/src/theme-files/index.js

// ...
import Post from "./Post";

const Root = ({ state }) => {
  const data = state.source.get(state.router.link);

  return (
    <>
      {/* ... */}
      <main>
        {data.isArchive && <List />}
        {data.isPost && <Post />}
        {data.isPage && <Post />}
      </main>
    </>
  );
};
```

## 9. Display the content of posts and pages separately

At the moment posts and pages both use the same component. But normally posts will display author and date information as well as tags, categories, etc...

Let's distinguish between the two now.

Create a new file and call it `page.js`. Copy the contents of `post.js` into `page.js` and rename the component.

```jsx
// File: /packages/jsnation-theme/src/theme-files/page.js

import React from "react"
import { connect } from "frontity"

const Page = ({ state }) => {
    const data = state.source.get(state.router.link)
    const post = state.source[data.type][data.id]

    return (
        <div>
            <h2>{post.title.rendered}</h2>
            <div dangerouslySetInnerHTML={{ __html: post.content.rendered}} />
        </div>
    )
}

export default connect(Page)
```

At the moment `page.js` and `post.js` are pretty much identical, so let's now distinguish between them by adding author and date info to `post.js`.

```jsx
// File: /packages/jsnation-theme/src/theme-files/post.js

import React from "react";
import { connect } from "frontity";

const Post = ({ state }) => {
  const data = state.source.get(state.router.link);
  const post = state.source[data.type][data.id];
  const author = state.source.author[post.author]

  return (
    <div>
      <h2>{post.title.rendered}</h2>
      <p><strong>Posted: </strong>{post.date}</p>
      <p><strong>Author: </strong>{author.name}</p>
      <div dangerouslySetInnerHTML={{ __html: post.content.rendered }} />
    </div>
  );
};

export default connect(Post);
```

Finally in this section let's change the root component to import and use the `<Page>` component.

```jsx
// File: /packages/jsnation-theme/src/theme-files/index.js

// ...
import Page from "./page"

const Root = ({ state }) => {
  const data = state.source.get(state.router.link);

  return (
    <>
      {/* ... */}
      <main>
        {data.isArchive && <List />}
        {data.isPost && <Post />}
        {data.isPage && <Page />}
      </main>
    </>
  );
};
```

## 10. Add some style





## Use state and actions
## Add tags to the <head>










---

> Get more info about [how to deploy](https://docs.frontity.org/deployment) a Frontity project

---

### » Frontity Channels 🌎

We have different channels at your disposal where you can find information about the project, discuss about it and get involved:

- 📖 **[Docs](https://docs.frontity.org)**: this is the place to learn how to build amazing sites with Frontity.
- 👨‍👩‍👧‍👦 **[Community](https://community.frontity.org/)**: use our forum to [ask any questions](https://community.frontity.org/c/dev-talk-questions), feedback and meet great people. This is your place too to share [what are you building with Frontity](https://community.frontity.org/c/showcases)!
- 🐞 **[GitHub](https://github.com/frontity/frontity)**: we use GitHub for bugs and pull requests. Questions are answered in the [community forum](https://community.frontity.org/)!
- 🗣 **Social media**: a more informal place to interact with Frontity users, reach out to us on [Twitter](https://twitter.com/frontity).
- 💌 **Newsletter**: do you want to receive the latest framework updates and news? Subscribe [here](https://frontity.org/)

### » Get involved 🤗

Got questions or feedback about Frontity? We'd love to hear from you. Use our [community forum](https://community.frontity.org) yo ! ❤️

Frontity also welcomes contributions. There are many ways to support the project! If you don't know where to start, this guide might help → [How to contribute?](https://docs.frontity.org/contributing/how-to-contribute)
