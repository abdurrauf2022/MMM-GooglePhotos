# MMM-GooglePhotos
Display your photos from album of Google Photos on MagicMirror

## Screenshot
![](https://raw.githubusercontent.com/eouia/MMM-GooglePhotos/master/sc.png)

![](https://raw.githubusercontent.com/eouia/MMM-GooglePhotos/master/sc2.png)

## Regular installation
1. Install Module
```sh
git clone https://github.com/eouia/MMM-GooglePhotos.git
cd MMM-GooglePhotos
npm install
```

If you doubt errors are caused by different `node.JS` version, you can try to compile it by yourself.
In that case, you need to rebuild some binaries to match with Electron version.
```sh
cd ~/MagicMirror/modules/MMM-GooglePhotos
npm install --save-dev electron-rebuild
./node_modules/.bin/electron-rebuild   # It could takes dozens sec.
```

2. If you are using Docker
```sh
cd ~/magic_mirror/modules
git clone https://github.com/eouia/MMM-GooglePhotos.git
docker exec -it -w /opt/magic_mirror/modules/MMM-GooglePhotos magic_mirror npm install
```


### Get `token.json`
1. Clone this repo in your local pc and execute `npm install`
2. Go to [Google API Console](https://console.developers.google.com/)
3. From the menu bar, select a project or create a new project.
4. To open the Google API Library, from the Navigation menu, select `APIs & Services > Library`.
	 Don't forget to enble the Google API Services.
5. Search for "Google Photos Library API". Select the correct result and click Enable. (You may need to enable "Google Plus" also.)
6. Then, from the menu, select `APIs & Services > Credentials`.
7. On the Credentials page, click `Create Credentials > OAuth client ID`.
8. Select your Application type as **`Desktop app`**(IMPORTANT!!!) and submit. (Before or After that, you might be asked for making consent screen. do that.)
> Google might change the menu name. So current this would work; ![2022-09-18_18-49-03](https://user-images.githubusercontent.com/2337380/190921355-49162763-0fdd-4b7e-a361-d762046f844d.png)

9. Then, you can download your credential json file from list. Downloaded file name would be `client_secret_xxxx...xxx.json`. rename it as `credentials.json` and save it to your `MMM-GooglePhotos` directory.
10. Now, open your termial
```shell
cd ~/MagicMirror/modules/MMM-GooglePhotos
node generate_token_v2.js
```
10. At first execution, It will open a browser and will ask you to login google account and to consent your allowance.
11. Authorize it and close the browser
12. Copy the file `token.json` and `credentials.json` to the folder `MMM-GooglePhotos` in the remote device

### stop token from expiring every week
-as of 2021, it appears tokens only last 1 week while in 'testing'. This led to users needing to get a new token.json every ~week to 10 days. To get your app out of testing mode, where the token will last indefinately:

1. go to your google cloud console, select your magic mirror project. Then from the navigation menu(top left) -> APIs & Services -> Oath consent screen. This should get you to a site something like https://console.cloud.google.com/apis/credentials/consent?project=[PROJECT_ID] where [PROJECT_ID] is the project ID. This is where the publishing status. It looks like this:

![](https://raw.githubusercontent.com/eouia/MMM-GooglePhotos/master/PublishAppScreen.png)

2. click Publish app and review permissions as necessary.


## Configuration
```javascript
{
  module: "MMM-GooglePhotos",
  position: "top_right",
  config: {
		albums: [], // Set your album name. like ["My wedding", "family share", "Travle to Paris"]
		updateInterval: 1000 * 60, // minimum 10 seconds.
		sort: "new", // "old", "random"
		uploadAlbum: null, // Only album created by `create_uploadable_album.js`.
		condition: {
			fromDate: null, // Or "2018-03", RFC ... format available
			toDate: null, // Or "2019-12-25",
			minWidth: null, // Or 400
			maxWidth: null, // Or 8000
			minHeight: null, // Or 400
			maxHeight: null, // Or 8000
			minWHRatio: null,
			maxWHRatio: null,
			// WHRatio = Width/Height ratio ( ==1 : Squared Photo,   < 1 : Portraited Photo, > 1 : Landscaped Photo)
		},
		showWidth: 1080, // These values will be used for quality of downloaded photos to show. real size to show in your MagicMirror region is recommended.
		showHeight: 1920,
		timeFormat: "YYYY/MM/DD HH:mm", // Or `relative` can be used.
  }
},
```

## Usage
#### **`albums`**
Now this module can access not only your owns but also shared. You can specify album title like this.
```js
albums: ["My wedding", "family share", "Travle to Paris", "from Tom"],
```
- Caution. Too many albums and photos could make long bootup delay.
- Remember this. You can only show max 8640 photos in a day. Manage your album what to show, it will make better performance.

#### **`updateInterval`**
- Minimum `updateInterval` is 10 seconds. Too often update could makes API quota drains or network burden.

#### **`sort`**
- `new`, `old`, `random` are supported.

#### **`uploadAlbum`**
- If you set this, you can upload pictures from MagicMirror to Google Photos through `GPHOTO_UPLOAD` notification.
```js
this.sendNotification('GPHOTO_UPLOAD', path)
```
- This album **SHOULD** be created by `create_uploadable_album.js`.
```sh
node create_uploadable_album.js MyMagicMirrorAlbum
```
- At this moment, `MMM-Selfieshot` and `MMM-TelegramBot` can upload their pictures through this feature.

#### **`condition`**
- You can filter photos by this object.
- `fromDate` : If set, The photos which was created after this value will be loaded. (e.g: `fromDate:"2015-12-25"` or `fromDate:"6 Mar 17 21:22 UT"`)
- `toDate` : If set, The photos which was created before this value will be loaded. (e.g: `toDate:"Mon 06 Mar 2017 21:22:23 z"` or `toDate:"20130208"`)
- ISO 8601 and RFC 2822 is supported for `fromDate` and `toDate`.
- `minWidth`, `maxWidth`, `minHeight`, `maxHeight` : If set, the photos have these value as original dimensiont will be loaded. You can use these values to avoid too big or too small pictures(like icons)
- `minWHRatio`, `maxWHRatio` : With these values, you can get only portrait photos(or landscaped, or squared)
- **WHRatio** is `width / height`. So `=1` will be squared dimension. `>1` will be landscaped. `<1` will be portrait.
- Example:
```js
condition: {
	fromDate: "2018-01-01", // I don't want older photos than this.
	minWidth: 600, // I don't want to display some icons or meme-pictures from my garbage collecting albums.
	maxWHRatio: 1, // I want to display photos which are portrait.
}
```

#### **`showWidth`, `showHeight`**
- Specify your real resolution to show.

#### **`timeFormat`**
- Specify time format for photo info. You can also use `relative` to show more humanized.

#### **`debug`**
- If set, more detailed info will be logged.

#### **`autoInfoPosition`**
- For preventing LCD burning, Photo info can be relocated by condition.
  	- `true` : automatically change position to each corner per 15 minutes.
		- `false` : not using.
		- callbackfunction (album, photo) : User can make his own position. It should return `[top, left, bottom, right]`
```js
autoInfoPosition: true, // or false

// User custom callback
autoInfoPosition: (album, photo)=> {
 return ['10px', '10px', 'none', 'none'] // This will show photo info top-left corner.
}

```



## Tip.
- Not to show photo info : Add this into your `css/custom.css`.
```css
#GPHOTO_INFO {
	display:none;
}
```

- To move photo info to other position (e.g: top-left corner): Add this into your `css/custom.css`.
```css
#GPHOTO_INFO {
  top: 10px;
  left: 10px;
  bottom: inherit;
  right: inherit;
}
```

- Not to show blurred Background : Add this into your `css/custom.css`.
```css
#GPHOTO_BACK {
	display:none;
}
```

- To cover whole region with image : Add this into your `css/custom.css`.
```css
#GPHOTO_CURRENT {
	background-size:cover;
}
```

- To shrink image and be fully visible on smaller screens : Add this into your `css/custom.css`.
```css
#GPHOTO_CURRENT {
	background-size:contain;
}
```

- To display `clock` more clearly on showing in `fullscreen_below` : Add this into your `css/custom.css`.
```css
.clock {
  padding: 10px;
	background-color: rgba(0, 0, 0, 0.5);
}
```

- To give opacity to photos:
```CSS
@keyframes trans {
  from {opacity: 0}
  to {opacity: 0.5}
}
#GPHOTO_CURRENT {
  background-size:cover;
  opacity:0.5;
}
```

## Notice
- First scanning will take a few (~dozens) seconds. Don't panic.
- If there are 1000s of photos, this scan could take minutes(over 10). longer scans increase the probablity of an error happening. If a single error happens in the scan, it will retry after 1 hour. After first successful scan, subsequent startups should go very quickly(seconds).
- 


## Last Tested;
- MagicMirror : v2.10.0
- node.js : required over v8.
