If you write code, I'm sure you've heard this like a million times but it's worth repeating: ***The best way to learn a new tool is to build something with it.***

So to welcome you in this journey of learning react-native, we will be building a camera interface, together. If you've used *Snapchat or Whatsapp*, the UI we will have by the end of this post should look familiar to you.

## I don't have time for this, show me the code
Hold on there, cowboy 🤠 I value your time so here's your code: [https://github.com/foysalit/rn-tutorial-vedo](https://github.com/foysalit/rn-tutorial-vedo). Hope you find solace in the rugged prairie :)

## Things You Should Already Know
This post is intended for beginners but you will need very basic understanding of javascript, es6, terminal, HTML, CSS and react to understand what's going on. I will try to link to documentations and resources wherever applicable but feel free to ask for additional resources/references if you think something isn't adding up with your understanding of things.

## Toolkit
React native is by itself to mobile development is similar to what HTML/CSS is to web development. Being very close to the metal, you will find yourself often scavenging for ready-made libraries on npm. This is where I should be telling you to learn the basics first, learn how to do things without 3rd party libraries etc. etc. Honestly though, I always thought you can learn a lot by just pulling down an existing library and using it first hand instead of reading 10 blog posts on how react native work at it's core. 

We will be using libraries like that wherever applicable throughout the building process of this app. 

## Flying Start
Starting off a React Native project is super easy and [expo](http://expo.io/) makes it even easier. Expo is like React Native on steriods. Have I sold you on it yet? Yeah? Great! Go ahead and install it [following their docs](https://docs.expo.io/versions/latest/introduction/installation).
To start our project, all we have to do is, come up with a name for our app (not as easy as it sounds). I've decided to call our app **vedo**. It means, *I see* in italian and since it's a camera app, I think that's an appropriate name for it. Now fire up your terminal and run the following command :

```bash
expo init vedo
```

This will prompt you to choose between 2 options. Expo can create a barebone react native app or an app with tab navigation implemented for you. For this project, we only need the barebone app so go ahead and select `blank` to finish off scaffolding.

At this point you should have the project folder `vedo/` created for you. Navigate inside that and run the following command to get the app started:

```bash
yarn start
``` 
This will fire up the Expo builder and output a QR code on your terminal. At this point, you need a device to run the app on before we can get to work. I personally like using an actual device during most of the development phase just because it feels really nice to actually handhold my app when building it. However, you're free to use simulator/emulator on your computer for development. [Expo's documentation](https://docs.expo.io/versions/v32.0.0/workflow/up-and-running#open-the-app-on-your-phone-or) can help you with that if you haven't setup your simulator yet. 

To see the app on your device, you need the Expo app on it. You can get it from the App Store or the Play Store depending on what device you have. 

I'm using an android and after downloading the Expo app on my phone, all I needed to do was, scan the QR code from the terminal and the app was running on my phone. How awesome is that!?

![up-and-running-with-expo.png](https://cdn.filestackcontent.com/nNl4QnSBQIuM0PxnZnm8)

## It's Coding Time
Building React Native app is an extremely pleasant experience. You can bring almost all your knowledge from react world and apply it in here. It makes it easy to hit the ground running with swiftly written code but it also helps you to write really modular code with components being the driving force. However, the purpose of this post is not to show you good architecture so we will let things get a little messy as long as it works and is easy to reason about.

Let's first create an `src/` folder in the root of the project and put 2 files in that folder: `camera.page.js` and `styles.js`. That's right, we have a `styles` file that ends with `.js`, the world is changing folks, get on with it!

Now, let's cleanup the App.js file in the root folder. We only need the following code in that file so replace all the junk you have in there and paste the following lines:

```avascript
// App.js file
import React from 'react';

import CameraPage from './src/camera.page';

export default class App extends React.Component {
    render() {
        return (
            <CameraPage />
        );
    };
};
```
> App.js file

Here we're importing the `CameraPage` component from the file we created earlier and rendering that in our `App` Component. Of course, this will throw a juicy error since our `camera.page.js` is still empty and does not return a react component. So let's open that up and put in a component so our terminal goes from red to green:

```es6
// src/camera.page.js file
import React from 'react';
import { View, Text } from 'react-native';
import { Camera, Permissions } from 'expo';

import styles from './styles';

export default class CameraPage extends React.Component {
    camera = null;

    state = {
        hasCameraPermission: null,
    };

    async componentDidMount() {
        const camera = await Permissions.askAsync(Permissions.CAMERA);
        const audio = await Permissions.askAsync(Permissions.AUDIO_RECORDING);
        const hasCameraPermission = (camera.status === 'granted' && audio.status === 'granted');

        this.setState({ hasCameraPermission });
    };

    render() {
        const { hasCameraPermission } = this.state;

        if (hasCameraPermission === null) {
            return <View />;
        } else if (hasCameraPermission === false) {
            return <Text>Access to camera has been denied.</Text>;
        }

        return (
            <View>
                <Camera
                    style={styles.preview}
                    ref={camera => this.camera = camera}
                />
            </View>
        );
    };
};
```
> src/camera.page.js file

A few things going on here but all of it is pretty much boilerplate stuff. Let's get through it piece by piece:

- We are importing a few modules from react-native, Expo and our `styles.js` file. I will explain each of them as they're used in the component code.
- We define a `camera` and `state` instance variable for our `CameraPage` component class. `camera` will hold a reference to the actual camera component that can be used to interact with the camera itself and give it instructions like take picture or record video etc. The `state` only has a `hasCameraPermission` property. As you may have seen, to access device camera from an app, the user needs to permit access to it and we use the state to keep track of the permission.
- We are using `componentDidMount` lifecycle component to request permissions from the user. Expo gives us a very handy `Permissions` module that can be used to request permission from users to access various features of the device. To access the camera we need `CAMERA` permission and to record audio within recorded video we need `AUDIO_RECORDING` permission. We request both using `Permission.askAsync` method. The `askAsync` method returns an object with the `status` property which is set to `granted` if the user accepts the request. We set `hasCameraPermission` to true only if both permissions are granted. Requesting permissions is a bit tricky and has a few edge cases that should be handled with better UX but for the purpose of this post, this will have to do.
- Now we move to the mighty `render()` method. As of now, What we show to the user depends on only one state variable, `hasCameraPermission`. Initially, we set it to be `null` remember? so if it's `null` that means user have neither denied nor granted permissions and we render an empty `<View/>`. Denying any of the permission prompts will set the `hasCameraPermission` to `false` and if that's the case, we render a simple text that tells the user that the permissions were denied. If none of the above cases were hit, we can safely assume that `hasCameraPermission` is set to `true` and we can try to render the camera view. This is where we use the `Camera` component imported from `expo` at the top of the file. Notice that we're setting `style={styles.perview}` which is the only thing we haven't defined yet. So let's write some styling, shall we?

> 💡**Pro Tip**: Users might deny permissions accidentally, so a full-proff UX would offer them a way to tell us to ask for permissions again. We are not gonna go into the nitty-gritty like that but it's definitely something you need to be aware of.

We will use the standard way of styling UI in React Native but know that there are several alternatives to this. If this is new to you, I'd recommend reading up on from the [official documentation](https://facebook.github.io/react-native/docs/stylesheet). 

React Native includes a `StyleSheet` and through module's `create` method, you can pass an object where each object where each property has another object assigned to them containing the actual styling. You can think of each property equivalent to a `css` selector and just like in css, you can define a styling for a class and apply the class for multiple html elements. 
OK, this is probably sounding more and more cryptic so let's look at some code at work. This is how our `styles.js` file looks like:

```javascript
// src/styles.js
import { StyleSheet, Dimensions } from 'react-native';

const { width: winWidth, height: winHeight } = Dimensions.get('window');

export default StyleSheet.create({
    preview: {
        height: winHeight,
        width: winWidth,
        position: 'absolute',
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
    },
});
```

Remember the `styles.preview` from our `CameraPage` component? This is where it came from. Our preview style basically says, make the camera component absolutely positioned and make it take up the full height and width of the device screen. To reliably set the height and width, we use the [`Dimensions` module](https://facebook.github.io/react-native/docs/dimensions) from `react-native`. `Dimensions.get('window')` returns an object with `width` and `height` properties containing the width and height of your device display, respectively. 

>💡**Pro Tip**: `width` and `height` as variable name is quite generic and easy to mix up with other variables in your code. Use es6 object destructuring to assign the `width` and `height` properties to a little more specific variable names. [Learn more about object destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).

## Preview Time
To me, one of the best feelings is writing a bunch of code and seeing the result of that on a screen. After 7 years of programming, I still get a rush of dopamine when I see my code rendering something on the screen. So, without further delay, let's see what we've got so far. While we've been writing code, Expo has been reloading our app on the device with the latest code but just to be sure, go ahead and stop the running Expo instance from the terminal by pressing `CTRL+C` and then restart it with `yarn start`. Then open your app again from Expo on your phone. It should ask you for the permissions, accept them in good faith and tadaaa 🎉 we have the camera showing up!!!
![permission-and-camera-preview.png](https://cdn.filestackcontent.com/vgu7QB8QFarzzsaVveGg)

Ok, may be it's not that big of a deal since we can't do anything with it .... yet.

## Reinventing The Camera
Expo's Camera module gives us access to almost all of the features of modern day phone cameras along with very advanced features like **face detection**. However, for a camera to work, first thing we need is a capture button. In addition, to demonstrate the use of some of the above advertised features, we will implement flash toggle and front-back camera switch. 

If you have built any UI for web or mobile, you know that a flexible grid system can be one of the most handy tools to have in your kit. React Native has amazing `flexbox` support but implementing a grid system with raw flexbox styling can be somewhat cumbersome. [`react-native-easy-grid` package](https://github.com/GeekyAnts/react-native-easy-grid) does a stellar job at that while being lightweight. 

Icons play a key role behind any modern, expressive and intuitive UI and the React Native community knows that very well which is why they've built [`react-native-vector-icons`](https://oblador.github.io/react-native-vector-icons/) which combines most of the amazing icon libraries on the web (ionicons, font-awesome, entypo etc.). Unfortunately, that package by itself is not directly compatible with Expo ***but worry not***, Expo has built `@expo/vector-icons` to make a bridge between them.

So let's pull these two libraries in with 

```console
yarn add react-native-easy-grid
yarn add @expo/vector-icons
```

To keep things a little more organized and clean, we will create a new component that holds the action buttons for the camera. Create a new file named `toolbar.component.js` in the `src/` folder and put the following piece of code in there:

```javascript
// src/toolbar.component.js file
import React from 'react';
import { Camera } from 'expo';
import { Ionicons } from '@expo/vector-icons';
import { Col, Row, Grid } from "react-native-easy-grid";
import { View, TouchableWithoutFeedback, TouchableOpacity } from 'react-native';

import styles from './styles';

const { FlashMode: CameraFlashModes, Type: CameraTypes } = Camera.Constants;

export default ({ 
    capturing = false, 
    cameraType = CameraTypes.back, 
    flashMode = CameraFlashModes.off, 
    setFlashMode, setCameraType, 
    onCaptureIn, onCaptureOut, onLongCapture, onShortCapture,  
}) => (
    <Grid style={styles.bottomToolbar}>
        <Row>
            <Col style={styles.alignCenter}>
                <TouchableOpacity onPress={() => setFlashMode( 
                    flashMode === CameraFlashModes.on ? CameraFlashModes.off : CameraFlashModes.on 
                )}>
                    <Ionicons
                        name={flashMode == CameraFlashModes.on ? "md-flash" : 'md-flash-off'}
                        color="white"
                        size={30}
                    />
                </TouchableOpacity>
            </Col>
            <Col size={2} style={styles.alignCenter}>
                <TouchableWithoutFeedback
                    onPressIn={onCaptureIn}
                    onPressOut={onCaptureOut}
                    onLongPress={onLongCapture}
                    onPress={onShortCapture}>
                    <View style={[styles.captureBtn, capturing && styles.captureBtnActive]}>
                        {capturing && <View style={styles.captureBtnInternal} />}
                    </View>
                </TouchableWithoutFeedback>
            </Col>
            <Col style={styles.alignCenter}>
                <TouchableOpacity onPress={() => setCameraType(
                    cameraType === CameraTypes.back ? CameraTypes.front : CameraTypes.back
                )}>
                    <Ionicons
                        name="md-reverse-camera"
                        color="white"
                        size={30}
                    />
                </TouchableOpacity>
            </Col>
        </Row>
    </Grid>
);
```

Hope that doesn't look too menacing. First of all, this is a [functional component](https://javascriptplayground.com/functional-stateless-components-react/). It does not concern itself with managing state or performing actions. it simply renders a view based on the props it receives and on various interactions it will delegate the events to it's parent renderer through function calls. Let's break it down and figure out what this is doing.

- We start by importing the components and modules we will be using from the third party libraries and our `styles.js` file. 
- The `Camera.Constants` object contains a number of key-value pairs to conveniently access various available modes and settings of the camera component. For our usecase we only need the `FlashMode` and `Type` for flash and front/back camera settings. Again, we're using object destructuring to assign them to a bit more meaningful variable names `CameraFlashModes` and `CameraTypes` maintaining their contexts.
- Our component accepts a number props that we will pass from the parent that renders this component. We will see how and when each of them are used within the component. We're creating a `Grid` with one `Row` that has 3 `Column` children. The `width` prop is used to set relative width of the columns. By setting `width={2}` we're telling the grid to make the middle column, twice as wide as the other column. So if you do the math: we have 3 columns, two of them have the same width and one of them has twice the width of each of the other two, rendering the left and right column to have `25%` and the middle column to have `50%` width of the container. If that made little sense to you, read up more on [flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/) and the [Easy Grid's official doc](https://github.com/GeekyAnts/react-native-easy-grid#react-native-easy-grid-). 
- The left and right most columns contain a button with an icon each and the middle column contains the capture button created using the `View` component. To give the buttons a little bit of interactive feedback, we wrap the icons in `TouchableOpacity` component from react-native. For icons, we're using `Ionicons` but you're free to choose any other icon providers. 
- The left button controls the flash mode. Pressing that button fires `setFlashMode` function that is passed as a prop. Also, based on the `flashMode` prop, it shows either `flash-on` or the `flash-off` icon.
- The right button controls if the back or the front camera is in use. So, when pressed it calls the `setCameraType` function so the parent component knows that the user has switched the camera from front to back or vice-versa.
- Now the capture button is a little more tricky. Mainly because we'll use this button for both recording video and capturing photos. On press and hold, the camera will start recording a video whereas on a short tap, it will only take a photo. Which is why instead of one single `onPress` event handler we have 4 different event handlers: `onPress`, `onPressIn`, `onPressOut` and `onLongPress` and since it's not a straight forward button, we don't want any immediate feedback from it so we wrap it in `TouchableWithoutFeedback` component. 
- The child component of the capture button is a bit more interesting. When `capturing` is true, it adds `captureBtnActive` style on top of `captureBtn` style and then it renders a child `View` component with `captureBtnInternal` style. How and when that `capturing` prop becomes true is another story and we'll get to it soon but this is a good place to start talking about the styling. We're using a bunch of styles coming from the `src/styles.js` file so let's see what these styles actually are. 

### Make it look good
Let's get back to our `src/styles.js` file and add the stylings for our camera toolbar buttons. We have a general purpose, utility style defined as `alignCenter`and then we have a few styles that are specific to specific elements such as `captureBtn`, `bottomToolbar` etc.

```javascript
// src/styles.js file
// ... previously written code
    alignCenter: {
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
    },
    bottomToolbar: {
        width: winWidth,
        position: 'absolute',
        height: 100,
        bottom: 0,
    },
    captureBtn: {
        width: 60,
        height: 60,
        borderWidth: 2,
        borderRadius: 60,
        borderColor: "#FFFFFF",
    },
    captureBtnActive: {
        width: 80,
        height: 80,
    },
    captureBtnInternal: {
        width: 76,
        height: 76,
        borderWidth: 2,
        borderRadius: 76,
        backgroundColor: "red",
        borderColor: "transparent",
    },
//... previously written code
```

- `alignCenter` horizontally and vertically centers all of an element's children. 
- `bottomToolbar` makes our entire toolbar full width of our device screen and positions it at the bottom of the screen.
- `captureBtn` is a circular button with white border by default which is a very common UI for the capture button of a on screen camera.
- `captureBtnActive` makes the button a bit larger in size when the user taps on the button and by making it bigger, we can make sure that the entire button isn't covered by user's finger. 
- `captureBtnInternal` renders a red circle inside the capture button to indicate that the camera is either recording a video or taking a picture.

With all that explained, it's time to see the toolbar component in action. Let's go back to our `src/camera.page.js` file:

```javascript
// src/camera.page.js

// ... previously written code
import Toolbar from './toolbar.component';

export default class CameraPage extends React.Component {
// ... previously written code
            <React.Fragment>
                <View>
                    <Camera
                        style={styles.preview}
                        ref={camera => this.camera = camera}
                    />
                </View>

                <Toolbar />
            </React.Fragment>
 // ...previously written code
```

We import the `Toolbar` component then add it underneath the previously rendered `<View>` and wrap everything in `React.Fragment`. React does not allow rendering multiple children without a parent wrapper. However, sometimes, to get proper layout, sometimes you may need to render multiple elements without a wrapper component and `React.Fragment` is used to do exactly that. Now, let's get back to our phone and you should see something like this:
![toolbar-preview.png](https://cdn.filestackcontent.com/PwElK3yTSRGdCCgUb8Vp)

Now, don't go tapping around the buttons and all cause it may look pretty but none of them actually do anything yet. Let's change that with a little bit of *state magic sprinkles* of react. Back to the `src/camera.page.js` file:

```javascript
// src/camera.page.js

// ... previously written code

	camera = null;
    state = {
        captures: [],
        flashMode: null,
        capturing: null,
        cameraType: null,
        hasCameraPermission: null,
    };

    setFlashMode = (flashMode) => this.setState({ flashMode });
    setCameraType = (cameraType) => this.setState({ cameraType });
    handleCaptureIn = () => this.setState({ capturing: true });

    handleCaptureOut = () => {
        if (this.state.capturing)
            this.camera.stopRecording();
    };

    handleShortCapture = async () => {
        const photoData = await this.camera.takePictureAsync();
        this.setState({ capturing: false, captures: [photoData, ...this.state.captures] })
    };

    handleLongCapture = async () => {
        const videoData = await this.camera.recordAsync();
        this.setState({ capturing: false, captures: [videoData, ...this.state.captures] });
    };

// ...previously written code 

    render() {
        const { hasCameraPermission, flashMode, cameraType, capturing } = this.state;

// ...previously written code

                <View>
                    <Camera
                        type={cameraType}
                        flashMode={flashMode}
                        style={styles.preview}
                        ref={camera => this.camera = camera}
                    />
                </View>

                <Toolbar 
                    capturing={capturing}
                    flashMode={flashMode}
                    cameraType={cameraType}
                    setFlashMode={this.setFlashMode}
                    setCameraType={this.setCameraType}
                    onCaptureIn={this.handleCaptureIn}
                    onCaptureOut={this.handleCaptureOut}
                    onLongCapture={this.handleLongCapture}
                    onShortCapture={this.handleShortCapture}
                />
// ... previously written code
```
> src/camera.page.js file

OK, bunch of code there, let's break down what's happening here:

- Our state now has `captures`, `flashMode`, `capturing` and `cameraType` properties. 
- `captures` will store all the photos and videos we will capture through the camera.
- `setFlashMode` and `setCameraType` methods simply updates the state with the values that are passed to them, and we already saw how they're called in our `Toolbar` component. 
- `handleCaptureIn` sets the `capturing` state to `true` and everytime the capture button is pressed, this will be triggered.
- `handleCaptureOut` attempts to stop recording video if `capturing` is set to `true` using the `stopRecording()` method. 
- `handleShortCapture` uses `takePictureAsync()` method of the camera component to take a photo and then it adds the returned data to the `captures` array and sets `capturing` to `false`. We will be using the `captures` array soon to display the captured videos and photos.
- In a similar fashion `handleLongCapture` uses the `recordAsync()` method of the camera component and tells the camera to start recording video. Reminder that `handleLongCapture` is called from the `Toolbar` component when user taps and holds the capture button. Then of course, we save the returned data in the `captures` array using es6 array spreading.
- Then we simply pass these state data and methods to the components that need them in our render method. 

That's it for the camera, I promise. However, to see that the camera is working, we need to visualize the photos and videos the camera takes.

## Gallery Component
Let's start by creating a `gallery.component.js` file in the `src/` folder and we will make this a stateless functional component too just like our toolbar component:

```javascript
// src/gallery.component.js file

import React from 'react';
import { View, Image, ScrollView } from 'react-native';

import styles from './styles';

export default ({captures=[]}) => (
    <ScrollView 
        horizontal={true}
        style={[styles.bottomToolbar, styles.galleryContainer]} 
    >
        {captures.map(({ uri }) => (
            <View style={styles.galleryImageContainer} key={uri}>
                <Image source={{ uri }} style={styles.galleryImage} />
            </View>
        ))}
    </ScrollView>
);
```

This component renders a horizontally scrollable gallery of all the images our camera takes using the `ScrollView` component from react-native. Each successful call to `takePictureAsync` and `recordAsync` methods of the camera component returns an object containing a property `uri` that refers to the photo/video captured/recorded and since we stored all of them in the `captures` array, we can iterate through each entry in that array and render an `Image` component with it's source pointing to the `uri`.

Now it also contains a few new style properties. So let's add those along with the other styles:

```javascript
// src/styles.js file

// ...  previously written code
    galleryContainer: { 
        bottom: 100 
    },
    galleryImageContainer: { 
        width: 75, 
        height: 75, 
        marginRight: 5 
    },
    galleryImage: { 
        width: 75, 
        height: 75 
    }
// ... previously written code
```

We're putting the gallery right above the `Toolbar` component. Then each of the image and their containers are given a square size of `75x75px` size with a `5px` gap between each photo. 

Ok, let's use the gallery in our `src/camera.page.js`:

```javascript
// src/camera.page.js

// ... previously written code

import Toolbar from './toolbar.component';
import Gallery from './gallery.component';

// ... previously written code

    render() {
        const { hasCameraPermission, flashMode, cameraType, capturing, captures } = this.state;
        
// ... previously written code

                {captures.length > 0 && <Gallery captures={captures}/>}

                <Toolbar 

// ... previously written code
```

We're simply importing the `Gallery` component and rendering it with `captures` property from the state if the user has captured any photo/video at all.

## Dopamine Time
Alright, Alright, Alright! It's time for that sweet sweet demo and dopamine rush. With all of the above, you should have something like below:

![Full Demo Gif](https://i.imgur.com/xXaFCQb.gifv)
> Full Demo of our app. [Watch it on youtube](https://youtu.be/rdxuR28Gsyo) or [if you prefer gifs](https://i.imgur.com/xXaFCQb.gifv)

Congratulations! You've done it! You're now the proud owner of the hip new camera app, Vedo...sorry, I tend to oversell things but hey, it's something. 
![Sorry not sorry](https://cdn.filestackcontent.com/OnTt9MNlQy2r6pzgxxR6)

## Homework
I know, I know... ain't nobody got time for that. However, I'd urge you to play around with the code. As an additional task, you can build a new feature to show the captured video/photo in fullscreen when the user taps on one. 

Feel free to ask any questions or leave a comment if you've found the post helpful. [Me on twitter](https://twitter.com/foysalit)

### Credits
1. [Cover Photo by Jason Blackeye on Unsplash](https://unsplash.com/photos/CtpETScvtJU?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)