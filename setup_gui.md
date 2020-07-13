[ &larr; back](setup_root.md)
<br/>
# Set up GUI
Once we have our models defined and our graphql server running, we can generate and run a graphical user interface.
To install it, we need to run the following lines:
### Install GUI skeleton
```
$ git clone https://github.com/ScienceDb/single-page-app.git gui-skeleton
$ cd gui-skeleton
$ npm install
```
From now on we will assume that your gui skeleton is stored in `/your-path/gui-skeleton`

### Install GUI generator

```
$ git clone https://github.com/ScienceDb/single-page-app-codegen.git gui-generator
$ cd gui-generator
$ npm install -g
```

### Generate GUI
```
gui-generator -h

 Usage: gui-generator [options]

 Code generator for SPA

 Options:
   -f, --jsonFiles <filesFolder>      Folder containing one json file for each model
   -o, --outputDirectory <directory>  Directory where generated code will be written
   -h, --help                         output usage information
   ```
Example:
```
$ gui-generator -f /your-path/json-files -o /your-path/gui-skeleton
```
### Environment variables

* `MY_SERVER_URL` - url where your backend server will be running, default value is `http://localhost:3000/graphql`
* `MY_LOGIN_URL` - url where your backend will check authentication, default value is `http://localhost:3000/login`.
* `MAX_UPLOAD_SIZE`- maximum size (in MB) of a file intended to be uploaded, default value is `500`, which means that users cannot upload a file larger than 500MB.

### Run GUI
```
cd /your-path/gui-skeleton
npm run dev
```
