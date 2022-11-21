# Node.js tips
### Access the default export on Dynamic Import()
shows.js:
````
const shows = [
    {
        id: '0',
        title: 'Show 1'
    }
]
export default shows;
````
The dynamic import() loads the module and return a module object that contains all its exports. 
In order to access the default export use the default property of the module object:
````
    const someFunction = async(src) => {
       if (process.env.USE_API === 'NO') {
          const { default: data } = await import(`@data/${src}`);
          console.log(data);
       }
    }
````
The output in the terminal will be:
````
[
  {
     id: '0',
     title: 'Show 1'
  }
]
````
