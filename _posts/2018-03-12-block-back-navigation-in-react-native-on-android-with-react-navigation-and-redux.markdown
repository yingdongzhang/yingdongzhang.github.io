---
layout: post
title:  "Block Back Navigation in React Native on Android with React Navigation and Redux"
date:   2018-03-12 21:00:00 +1000
categories: [post,Javascript,React Native]
comments: true
---

## The default setup

If you are building a React Native application using [React Navigation](https://reactnavigation.org/) and Redux, you will need to add a global handler for the Android hardware back button. You would put it on your top-level component, such as the `App.js` component as the following:

```Javascript
import { BackHandler } from 'react-native'
import { NavigationActions } from 'react-navigation'
class ReduxNavigation extends React.Component {
  componentDidMount() {
    BackHandler.addEventListener("hardwareBackPress", this.onBackPress)
  }
  componentWillUnmount() {
    BackHandler.removeEventListener("hardwareBackPress", this.onBackPress)
  }
  onBackPress = () => {
    const { dispatch, nav } = this.props
    if (nav.index === 0) {
      return false;
    }
    dispatch(NavigationActions.back())
    return true;
  };

  render() {
    //   ...
  }
}
```

In `componentDidMount` you add a event listener to the `hardwareBackPress` event, which calls the `onBackPress` function. 

If you are not at the root index (nav.index === 0), you then dispatch a back action, and return true to stop the app from exiting, otherwise return false to exit the app.

Then you remove the event listener inside `componentWillUnmount` so there won't be memory leak.

## The problem

That was all easy and simple. However, what if on a pariticular screen you want to handle the back button press a bit differently. For example, on a screen you have a form, and you want to alert the user when they tap the back button if the form is dirty. You will notice that the default setup won't work because it doesn't perform such check.

## The solution

Let's say the screen you want to have a custom back button handler is called Form. We want to display an alert when the user taps the back button and the form is dirty. We will just do the same setup in the Form component, add a custom `hardwareBackPress` event handler inside `componentDidMount` as the following:

```Javascript
  componentDidMount() {
    BackHandler.addEventListener('hardwareBackPress', this.onBackPress)
  }
  componentWillUnmount() {
    BackHandler.removeEventListener('hardwareBackPress', this.onBackPress)
  }
  onBackPress = () => {
    const { goBack } = this.props // eslint-disable-line
    const { isDirty } = this.state // eslint-disable-line react/prop-types
    if (isDirty) {
      Alert.alert(
        'Unsubmitted Changes',
        'You have unsubmitted form changes that will be lost if you go back, are you sure you want to leave?',
        [
          { text: 'Cancel', style: 'cancel' },
          {
            text: 'OK',
            onPress: () => {
              dispatch(NavigationActions.back())
              return true
            },
          },
        ],
        { cancelable: false },
      )
      return false
    }
    dispatch(NavigationActions.back())
    return true
  }
```

In the custom back button handler we check if the Form is dirty, and if it is we will display the alert and only navigate back when the user taps "OK".

## The tricky bit

Now if you try it out you will notice that the alert does get shown, but it still navigates back. Why? Because remeber the top-level component is still alive, it is just hidden in the navigation stack, and its back handler also listens to the event. I firstly tried using React Navigation's lifecycle hooks `willBlur` and `didBlur`, but they didn't work 100% of the times.

This is where we can get a little help from Redux. We can add a state property to our Redux store called `globalBackHandlerEnabled`, which defaults to `true`. The idea is that in the custom `onBackPress` function inside the Form component, we will set the `globalBackHandlerEnabled` to `false` when we enter the screen, and in the global back button press handler, we can check the value of `globalBackHandlerEnabled`, and return right away if its value is `false`, otherwise we will continue. Then when the Form screen unmounts, we set `globalBackHandlerEnabled` back to true, and our global back button hanlder will be active again.

Just a note you will need to implement an action creator and a reducer to toggle the value of `globalBackHandlerEnabled` in your Redux store, I will skip this part as it would be the same with other Redux goodies.

In the global back button handler, we will just add the following lines at the very top of the function:

```Javascript
const { globalBackHandlerEnabled } = this.props
if (!globalBackHandlerEnabled) return true
```

Then in the Form component, we will toggle the status of the `globalBackHandlerEnabled` state property:

```Javascript
  componentDidMount() {
    this.props.onSetGlobalBackHandlerStatus(false)
    BackHandler.addEventListener('hardwareBackPress', this.onBackPress)
  }
  componentWillUnmount() {
    this.props.onSetGlobalBackHandlerStatus(true)
    BackHandler.removeEventListener('hardwareBackPress', this.onBackPress)
  }
```

That's it, now it will prevent the back navigation when the user tries to navigate away from a dirty form while still maintain the default behaviour.
