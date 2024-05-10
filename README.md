# Frazier Handmade Goods API Guide

A living document describing the currently-available features of the Frazier Handmade Goods API, as well as suggesting best practices for integration with the e-commerce storefront.

>💡 This file usually lives in a private repo for the Frazier Handmade Goods e-commerce site, but has been copied here for use as a writing sample! The Frazier Handmade Goods repo is divided into server and client folders, where `server` contains the API routes and `client` contains code for the front-end. This guide is usually included in the server folder, where it can easily be found by anyone who needs to use the API<br/>
>![The file structure of the Frazier Handmade Goods repo, showing a client, node_module, and server folders. A readme.md file is nested inside the server folder and circled in red](<fileStructure.png>)

## Table of Contents
  - [Getting Started](#getting-started)
  - [Data Structure](#data-structure)
    - [Requirements for Certain Values:](#requirements-for-certain-values)
      - [`store`](#store)
      - [`item_type`](#item_type)
      - [`imgs`](#imgs)
      - [`cost`](#cost)
      - [`addl`](#addl)
    - [Automatically Generated Values](#automatically-generated-values)
  - [API Endpoints](#api-endpoints)
    - [Create a New Item](#create-a-new-item)
    - [Update and Delete an Item](#update-and-delete-an-item)
    - [Reading Item Data](#reading-item-data)
      - [Pre-made Read Filters](#pre-made-read-filters)
  - [Using the Blob Store for Images](#using-the-blob-store-for-images)
    - [Adding a New Image - Walkthrough](#adding-a-new-image---walkthrough)
    - [Integration with Item Creation](#integration-with-item-creation)
  - [Additional Help](#additional-help)

## Getting Started

The Frazier Handmade Goods API is currently still in development. This means you'll have to make sure your packages are all up to date, and you'll have to run the server manually. If you've never worked in the repo and would like a step-by-step for setup, keep reading here. Otherwise, you can skip to the [Data Structure](#data-structure) or [API Endpoints](#api-endpoints) section.

1. Clone the repo

    ```git clone https://github.com/rhysrfrazier/FrazierKnivesFull.git```

    Alternatively, if you want to use SSH instead of HTTPS:

    ```git clone git@github.com:rhysrfrazier/FrazierKnivesFull.git```

2. Open a new feature branch to work in

    ```git checkout -b yourBranchName```

3. Open the terminal and cd into the server folder
   
   ```cd server```

4. Install dependencies
 
    ```npm install```<br>
    or <br>
    ```yarn install```
5. Run the server

    ```npm run dev```<br>
    or<br>
    ```yarn dev```

That's it! The server should be running on `http://localhost:3001`. You can either go to that URL in your web browser or run a GET request in your favorite REST API client to make sure it's running - either way, you should get a 200 status and a response that says "ok."

## Data Structure

The API is used to manage the Items table within the Frazier Handmade Goods database. The items model itself is flexible enough to accommodate any kind of product through the use of the ```addl``` key. The generic shape of an Item's data is as follows:

```
{
    "id": integer, 
    "store": string,
    "item_type": string,
    "title": string,
    "imgs": text array,
    "full_desc": text,
    "materials": string array,
    "cost": integer,
    "addl": object,
    "sold": boolean,
    "createdAt": Date,
    "keywords": string array,
    "care_instructions": text
}
```

For example, a GET request may return the following array:
```JSON
[{
    "id": 113,
    "store": "creations",
    "item_type": "earring",
    "title": "floral glass bead dangle earrings",
    "imgs": ["https://i.imgur.com/9h61J58.jpeg", "https://i.imgur.com/OS5bUlU.jpeg"],
    "full_desc": "This is a pair of test earrings, made of nothing, used for test purposes. They do not exist.",
    "materials": [
      "nothing",
      "air",
      "hopes and dreams"
    ],
    "cost": 1,
    "addl": {
      "colors": [
        "clear",
        "red",
        "blue"
      ],
      "category": "dangle"
    },
    "sold": false,
    "createdAt": "2024-04-03T19:56:32.400Z",
    "keywords": [
      "earring",
      "dangly",
      "dangle",
      "clear",
      "red",
      "blue",
      "nothing",
      "air",
      "hopes",
      "dreams"
    ],
    "care_instructions": "No special care needed"
  }]
```

### Requirements for Certain Values:

Certain values within the Items object have special requirements that must be taken into account for merchandise to be displayed properly in the store. The admin dashboard used by store owners accounts for this already; however, in the case that a developer wishes to seed Items into the database directly, or in the case that the store owners want to incorporate new kinds of products, it is important to understand how certain keys are used:

#### `store`

The `store` key must always have a value of either `"knives"` or `"creations"` in order to be displayed

#### `item_type`

Currently integrated values include `"knife"`, `"sheath"`, `"earring"`, `"wood turning"`, `"bracelet"`, and `"pottery"`. Others can be added, but will require store-front updates and RESTful API route updates.

#### `imgs`

Contains images for the item. `imgs` should be an array of strings, each containing the URL of an image.

Store owners could upload images to a site like [Imgur](https://imgur.com/), and use the image URLs generated by the site, but that would mean they have to use two different sites every time they want to upload an image.

To simplify how store owners will add images, we strongly recommend using the [blob store](#using-the-blob-store-for-images).

#### `cost`

JavaScript performs mathematic operations most accurately with integers, so `cost` must be recorded in cents - e.g., if a product is $60.00, it the value of `cost` should be the integer 6000.

|   ✅ Do this:   |               🚫 Not this:               |
| :------------: | :-------------------------------------: |
| `"cost": 6000` | `"cost": 60.00` <br> `"cost": "$60.00"` |

#### `addl`

`addl` is a nested object that allows us to specify properties that are particular to each `item_type`. Maintaining consistency in `addl` key-value pairs ensures we can correctly show the details of each item in the store page. Current requirements, based on the already existent `item_types`, are as follows:

- **`knife`**
    ```
    "addl": {
        "hardness": string, 
        "length": string, 
        "blade_length": string,
        "category:" string, 
    }
    ```
    - `hardness` should be a number on the Mho's Hardness Scale - it's a string because we don't need to do calculations with it, though (e.g. "8.70")
    - `length` should be the total length of the knife, with units included (e.g. "8 inches")
    - `blade+length` should be the length of just the blade, with units included (e.g. "4 inches")
    - `category` is any string that describes the kind of knife - no specific requirements. (e.g. "everyday carry", "hunting knife", etc.)
  
 - **`sheath`** <br>
    Sheaths include their inner dimensions. This means they can be sold separately from knives if needed, and customers will be able to determine which one will fit their knife.
    ```
    "addl": {
      "inner_length": string,
      "inner_width": string,
      "color": string,
      "style": string
    }
    ```
    - `inner_length` should be the greatest length of blade that could fit inside the sheath, with units (e.g. "4 inches")
    - `inner_width` should be the greatest width of blade that could fit inside the sheath, with units (e.g. "1.5 inches")
    - `color` is the dye color used for the sheath (e.g. "tan", "dark brown", etc.)
    - `style` is a description of any pattern or stamp used (e.g. "fish stamp", "basket-weave", etc.)

 - **`earring`** 
    ```
    "addl": {
      "colors": string array,
      "category": string
    }
    ```
    - `colors` should be an array of strings, e.g. ["blue", "purple", "clear"]
    - `category` should be a string that describes the kind of earring( e.g. "dangle", "stud", etc.)

 - **`wood_turning`** and **`pottery`** <br>
    Since both wood turnings and pottery are similar objects, they share requirements for the `addl` key. `addl` for each should contain length, width, and height, given as strings with units included, like so:
    ```
    "addl": {
      "length": "8 inches",
      "width": "8 inches",
      "height": "8 inches"
    }
    ```
 - **`bracelet`** 
    ```
    "addl": {
      "colors": string array,
      "clasp_type": string, 
    }
    ```
    - `colors` should again be an array of strings, e.g. ["blue", "purple", "clear"]
    - `clasp_type` should be a the kind of clasp, e.g. "lobster claw", "spring ring", "barrel", etc.
  
### Automatically Generated Values

Items have three keys whose values are automatically generated. This means you don't have to worry about adding certain keys to the JSON request body when making a POST request to create a new item.
- `id` is automatically generated and is sequential.
- `createdAt` is automatically generated based on the timestamp of item creation.
- `sold` is automatically set to `false` when a new item is created

## API Endpoints

The API has endpoints for full CRUD functionality, with a number of helpful options to get items that are pre-filtered in ways that will be helpful on the front-end.

> 💡 Keep in mind that API_BASEURL is currently `http://localhost:3001`. Once the database is deployed to a production environment this document will be updated with a new API_BASEURL.

### Create a New Item

To create a new item, make a POST request to `API_BASEURL/items` with JSON data for your new item in the request body. For example, if you're using the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API), your code may look something like this:

```JS
async function addItem(dataExample) {
    try{
        const response = await fetch("API_BASEURL/items", {
            method: "POST",
            mode: 'cors',
            body: JSON.stringify(dataExample)
        })
        const result = await response.json()
        console.log("Success:", result)
    } catch (err) {
        console.log("Error:", err)
    }
}

const dataExample = {
    "store": "creations",
    "item_type": "earring",
    "title": "floral glass bead dangle earrings",
    "imgs": ["https://i.imgur.com/9h61J58.jpeg", "https://i.imgur.com/OS5bUlU.jpeg"],
    "full_desc": "This is a pair of test earrings, made of nothing, used for test purposes. They do not exist.",
    "materials": [
      "nothing",
      "air",
      "hopes and dreams"
    ],
    "cost": 6000,
    "addl": {
      "colors": [
        "clear",
        "red",
        "blue"
      ],
      "category": "dangle"
    },
    "keywords": [
      "earring",
      "dangly",
      "dangle",
      "clear",
      "red",
      "blue",
      "nothing",
      "air",
      "hopes",
      "dreams"
    ],
    "care_instructions": "No special care needed"
}

addItem(dataExample)

```

> 💡 NOTE: `id`, `createdAt`, and `sold` are all part of the [data structure](#data-structure), but they [generate values automatically](#automatically-generated-values) when you create a new item. Don't include them in the request body of a POST request!

### Update and Delete an Item

Both PUT and DELETE requests are made to `API_BASEURL/item/[id]`, where [id] is the numerical id of the item you want to change or delete. For example: `API_BASEURL/item/2` would allow you to update or delete the item with the id of 2.

**An example of updating an item using the Fetch API:**

```JS
async function updateItem(data) {
    try{
        const response = await fetch("API_BASEURL/items/2", {
            method: "PUT",
            mode: 'cors',
            body: JSON.stringify(data)
        })
        const result = await response.json()
        console.log("Success:", result)
    } catch (err) {
        console.log("Error:", err)
    }
}

const exampleData = {
    "title": "new item title"
}
```

**An example of deleting an item using the Fetch API:**
```JS
async function deleteItem() {
    try{
        const response = await fetch("API_BASEURL/items/2", {
            method: "DELETE",
            mode: 'cors',
        })
        const result = await response
        console.log(result)
    } catch (err) {
        console.log("Error:", err)
    }
}
```
### Reading Item Data

You can get data from all unsold items in the database by using the `API_BASEURL/items` with a GET request. If you want to get information about a specific item, you can also use `API_BASEURL/items/[id]`.

**Fetch API Example:**
```JS
async function getItem() {
            const rawResponse = await fetch(`API_BASEURL/items` as string, {
                mode: 'cors'
            })
            const formattedResponse = await rawResponse.json()
            console.log(formattedResponse)
        }
```
>💡 NOTE: `API_BASEURL/items` and `API_BASEURL/items/[id]` do NOT include items that have a value of `sold: true`. The only way to access sold items is to use the endpoints `API_BASEURL/sold` and `API_BASEURL/sold/[id]`

#### Pre-made Read Filters

This API has a number of endpoints that allow you to get item data based on certain attributes, such as store or item type: 

| Endpoint                       | Purpose                                                                  |
| ------------------------------ | ------------------------------------------------------------------------ |
| `API_BASEURL/new`              | returns the 4 most recently-added items                                  |
| `API_BASEURL/sold`             | returns all items that have been marked as sold with `"sold": true`      |
| `API_BASEURL/sold/[id]`        | returns an item that's been marked as sold and has the specified item id |
| `API_BASEURL/frazierknives`    | returns all unsold items with `"store": "knives"`                        |
| `API_BASEURL/fraziercreations` | returns all unsold items with `"store": "creations"`                     |
| `API_BASEURL/knives`           | returns all unsold items with `"item_type": "knife"`                     |
| `API_BASEURL/sheaths`          | returns all unsold items with `"item_type": "sheath"`                    |
| `API_BASEURL/earrings`         | returns all unsold items with `"item_type": "earring"`                   |
| `API_BASEURL/bracelets`        | returns all unsold items with `"item_type": "bracelet"`                  |
| `API_BASEURL/pottery`          | returns all unsold items with `"item_type": "pottery"`                   |
| `API_BASEURL/wood_turnings`    | returns all unsold items with `"item_type": "wood turning"`              |
  
## Using the Blob Store for Images
Integrating the Blob Store for Item images has the potential to greatly simplify how store owners will add new Items to the Frazier Handmade Goods database. Instead of uploading images to [Imgur](https://imgur.com/), getting each individual URL, and then navigating back to the admin dashboard to input each URL into the form, they will be able to simply upload images directly through the admin dashboard.

Recall that the `imgs` key in the Items object is an array of strings, which should each be the URL of an image. That means access to image URLS is required _before_ attempting to create a new Item. Creating a new Item will therefore need at least two API calls - one as a POST request to the Blob Store, and another that uses the URLs from the Blob Store as values in the `imgs` key to make a POST request to the Frazier Handmade Goods API.

We'll start with a basic example of how to add an Image to the Blob Store, then talk about best practice for reducing the number of API calls we have to make when adding images to new Items.

### Adding a New Image - Walkthrough

Vercel supports a variety of different ways to access the Blob Store and upload a new file, but we'll focus on the most straightforward way here. For the sake of demonstration, let's assume that you're working in a client component to create a basic image uploader. Your boilerplate will probably look something like this if you're not using any additional libraries:

```jsx
`use client`;

export function imageUploaderExample() {

  return (
    <>
      <form>
        <label htmlFor='img'>
          Add an image:
          <input name='img' id='img' type='file' required/>
        </label>
        <button type='submit'>Upload</button>
      </form>
    </>
  )
}

```
<br/>

First, let's add the imports we'll need at the top of the file. For this example, we're just going to use some React Hooks (which we'll also call at the top of the component function) and the `put` method from `@vercel/blob`:
   <br/>

``` js
`use client`;
import { useState, useRef } from 'react';
import { put } from '@vercel/blob';

export function imageUploaderExample() {

const inputFileRef = useRef()
const [blob, setBlob] = useState()

  return(
    <>
      <form>
        <label htmlFor='img'>
          Add an image:
          <input name='img' id='img' type='file' ref={inputFileRef} required/>
        </label>
        <button type='submit'>Upload</button>
      </form>
    </>
  )
}
  
```


  <br/>

After importing `useRef`, we assign it to the `inputFieldRef` variable, and pass that variable to the `ref` prop on the input. This lets us access whatever value (in this case, whatever file) a user enters into the input field. For more information about the `useRef` hook, check out the [useRef docs](https://react.dev/reference/react/useRef) from React.

We use `useState` so React knows to re-render any relevant components when the variable changes. For example, if the user inputs file_A.png, then changes their mind and inputs file_B.png instead, React will know to update a preview component if you have one. Check out the [useState docs](https://react.dev/reference/react/useState) for more information.
  
Finally, the `put` method will allow us to upload a blob object to our Blob Store. It will take three required arguments:

1. `pathname`: a string specifying the base value of the return URL. For ours, we'll use the name of the file we're uploading.
2. `body`: the file itself. A blob object
3. `options`: a JSON object that allows us to specify certain parameters. For our purposes, we only need `access` and `token`. `access` must be set to `public` since that's all Vercel supports currently. `token` should be set to `process.env.BLOB_READ_WRITE_TOKEN`, which accesses the environment variable that connects to the Blob Store for this project.

For more information about the `put` method, check out [Vercel's docs](https://vercel.com/docs/storage/vercel-blob/using-blob-sdk#put).

Now that we have everything we need to access the user's file input, we'll start making a function that submits that input and pass it on to the `onSubmit` prop of the form component. This function will need to do a few things:

  - Prevent the page from reloading on submit. Otherwise, the page will re-render and we'll lose the file input before the upload to the database is complete.
  - Add the file to the Blob Store using the `put` method. This will take a moment, and we'll want to confirm that it works, so we'll need the function to be asynchronous
  - Set the `blob` state variable using `setBlob` so the user will still be able to see what file they just uploaded

>💡NOTE: if you're using TypeScript instead of JavaScript, you'll also need to acknowledge possible attempts to submit nothing. Our input has the `required` prop currently, which will block attempts to submit nothing for us; however, our submit function has no way of knowing that. TypeScript will throw an error saying `inputFileRef.current.files` is possibly null, so it's recommended you use a conditional to account for that. See the commented code in the following example for reference.

Once we build this out in it's most basic form, we'll have something like this:

```js
`use client`;
import { useState, useRef } from 'react';
import { put } from '@vercel/blob';

export function imageUploaderExample() {

const inputFileRef = useRef()
const [blob, setBlob] = useState()

const uploadImg = async (event) => {
  event.preventDefault() //1. prevent the page from reloading

  try {
    //2. add to the Blob Store
    const file = inputFileRef.current.files[0]; //console.log(inputFileRef) if you want to see where this comes from :)
    const newBlob = await put(file.name, file, {
      access: 'public',
      token: process.env.BLOB_READ_WRITE_TOKEN
    })

    //3. set blob state
    setBlob(newBlob)
    alert('uploaded successfully!')

  } catch (error) {
      console.log(error)
  }

  //EXAMPLE FOR TYPESCRIPT:
  // try {
  //   if (inputFileRef?.current?.files) {
  //     const file = inputFileRef.current.files[0];
  //     const newBlob = await put(file.name, file, {
  //       access: 'public',
  //       token: process.env.BLOB_READ_WRITE_TOKEN
  //     });

  //     setBlob(newBlob);
  //     alert('uploaded successfully')
  //   }
  // } catch (error) {
  //     console.log(error)
  // }
}

  return(
    <>
      <form onSubmit={uploadImg}>
        <label htmlFor='img'>
          Add an image:
          <input name='img' id='img' type='file' ref={inputFileRef} required/>
        </label>
        <button type='submit'>Upload</button>
      </form>
    </>
  )
}
```

<br/>

Test it out! You should get an alert letting you know your image has been successfully added to the Blob Store.

### Integration with Item Creation

Recall from the [data structure](#data-structure) of Items that the `imgs` key needs to be an array of image URLs. These URLs can be accessed on the return body of the `put` method, which we can see if we `console.log(newBlob)` in the code that added an image to the Blob Store:

![An image of the console, showing the request body with contentDisposition, contentType, downloadUrl, pathname, and url keys and their attendant values. url is circled in red](<consoleLog.png>)

We can avoid having to make another API call to the Blob Store to read this data if we take the URL directly from the request body and add it to the `imgs` array of the new Item we want to add to the database. This can be accomplished with a two-part form.

Let's look at an example to illustrate the general idea. First, let's add a state variable to the `imageUploaderExample` component to store all of the urls from newly uploaded images. We'll update that variable every time the user uploads a new picture to the blob:

```js
`use client`;
import { useState, useRef } from 'react';
import { put } from '@vercel/blob';

export function imageUploaderExample() {

const inputFileRef = useRef()
const [blob, setBlob] = useState()
const [urls, setUrls] = useState([]) //an array to hold all new image urls in state

const uploadImg = async (event) => {
  event.preventDefault()
  try {
    const file = inputFileRef.current.files[0];
    const newBlob = await put(file.name, file, {
      access: 'public',
      token: process.env.BLOB_READ_WRITE_TOKEN
    })
    setBlob(newBlob)
    setUrls([...urls, newBlob.url]) //updating the urls state variable
    alert('uploaded successfully!')
  } catch (error) {
      console.log(error)
  }
}

  return(
    <>
      <form onSubmit={uploadImg} >
        <label htmlFor='img'>
          Add an image:
          <input name='img' id='img' type='file' ref={inputFileRef} required/>
        </label>
        <button type='submit'>Upload</button>
      </form>
    </>
  )
}
```
<br/>

Next, we'll add a secondary form  like so:

1. Add a `done` state variable that indicates whether a user is finished uploading images. This variable will initially be set to `false`
2. Add a "Done with Image Upload" button that changes the `done` state variable to true when a user clicks it.
3. Conditionally render a secondary form based on the value of `done`. We won't write out an entire complex form here, but assume this is where we're getting all of the data for a new Item.

```js
`use client`;
import { useState, useRef } from 'react';
import { put } from '@vercel/blob';

export function imageUploaderExample() {

const inputFileRef = useRef()
const [blob, setBlob] = useState()
const [urls, setUrls] = useState([])
const [done, setDone] = useState(false) // 1. Add a done state variable and set it to false

const uploadImg = async (event) => {
  event.preventDefault()
  try {
    const file = inputFileRef.current.files[0];
    const newBlob = await put(file.name, file, {
      access: 'public',
      token: process.env.BLOB_READ_WRITE_TOKEN
    })
    setBlob(newBlob)
    setUrls([...urls, newBlob.url])
    alert('uploaded successfully!')
  } catch (error) {
      console.log(error)
  }
}

  return(
    <>
      <form onSubmit={uploadImg}>
        <label htmlFor='img'>
          Add an image:
          <input name='img' id='img' type='file' ref={inputFileRef} required/>
        </label>
        <button type='submit'>Upload</button>
      </form>
      {/* 2. Add a "Done with Image Upload button that changes the done state to true */}
      <button onClick={() => setDone(true)} >Done with Image Upload</button>
      {/* 3. Conditionally render a second form to gather Item data */}
      {done? 
        <form>
          {/* Get the rest of the Image data here */}
          <button type='submit'>Add Item</button>
        </form>
      : null}
    </>
  )
}
```

<br/>

Finally, we'll add a mock state variable to show where data from the Item upload form might be stored and handled, and add a function called `addItem()` that will run on submit. This function will store a snapshot of the `data` object in a new variable called `newItem,`, add an additional key called `imgs` and give it the value of our `urls` state variable, and then make a `POST` request to our database with `newItem` in the body:

```js
`use client`;
import { useState, useRef } from 'react';
import { put } from '@vercel/blob';

export function imageUploaderExample() {

const inputFileRef = useRef()
const [blob, setBlob] = useState()
const [urls, setUrls] = useState([]) 
const [done, setDone] = useState(false)
const [data, setData] = useState({})

const uploadImg = async (event) => {
  event.preventDefault()
  try {
    const file = inputFileRef.current.files[0];
    const newBlob = await put(file.name, file, {
      access: 'public',
      token: process.env.BLOB_READ_WRITE_TOKEN
    })
    setBlob(newBlob)
    setUrls([...urls, newBlob.url]) 
    alert('uploaded successfully!')
  } catch (error) {
      console.log(error)
  }
}

//new addItem() function:
const addItem = (data) => {
    const newItem = data
    newItem.imgs = urls
    try{
        const response = await fetch("API_BASEURL/items", {
            method: "POST",
            mode: 'cors',
            body: JSON.stringify(newItem)
        })
        const result = await response.json()
        console.log("Success:", result)
    } catch (err) {
        console.log("Error:", err)
    }
}

  return(
    <>
      <form onSubmit={uploadImg} >
        <label htmlFor='img'>
          Add an image:
          <input name='img' id='img' type='file' ref={inputFileRef} required/>
        </label>
        <button type='submit'>Upload</button>
      </form>
      <button onClick={() => setDone(true)} >Done with Image Upload</button>
      {done? 
        <form onSubmit={() => addItem(data)}>
          {/* Get the rest of the Image data here, use it to set the data state variable */}
          <button type='submit'>Add Item</button>
        </form>
      : null}
    </>
  )
}
```

<br/>

Whether you use a form component library or crete your own forms from scratch, you should be able to follow this general process to use the urls from the blob return body for the Item `imgs` keyword.

## Additional Help

If you have any questions, feel free to email me at <rhysrfrazier@gmail.com>
