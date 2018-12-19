# Set up GUI
Once we have our models defined and our graphql server running. We can generate and run a graphic user interface.
In order to install we need to run the following lines:
### Install GUI skeleton
```
$ git clone https://github.com/ScienceDb/ScienceDbGui.git gui-skeleton
$ cd gui-skeleton
$ git checkout enciclovida
$ npm install
```
From now on we will assume that your gui skeleton is stored in `/your-path/gui-skeleton`

### Install GUI generator

```
$ git clone https://github.com/ScienceDb/admin_gui_gen.git gui-generator
$ cd gui-generator
$ git checkout Features
$ npm install -g
```

### Generate GUI
```
 $ admin_gui_gen [options] <directory>

  <directory> path where GUI components will be rendered
  Options:
    --jsonFiles <files-directory>      Directory containing one json file for each model.
    -h, --help                         Output usage information
```
Example:
```
$ admin_gui_gen --jsonFile /your-path/json-files /your-path/gui-skeleton
```
### Check global variables
Expected environmental variables are
```
AUTH0_DOMAIN
AUTH0_CLIENT_ID
YOUR_CALLBACK_URL
MY_SERVER_URL
```
First three variables will be the ones provided by your auth0 configuration. For more details on how to cofigure your app with auth0 see [HERE](https://auth0.com/docs/quickstart/spa/vuejs#configure-auth0)

`MY_SERVER_URL` is the url where your backend server will be running

### Run GUI
```
cd /your-path/gui-skeleton
npm run dev
```
