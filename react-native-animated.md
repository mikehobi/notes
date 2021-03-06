# Tips for React Native Animated

I've been working with React Native's Animated components for almost two years, and I have a few things to share that have helped my workflow.

The documentation for Animated can be found here: https://facebook.github.io/react-native/docs/animated.html

This provides details about the basic API, but I would like to go more in-depth and propose a few patterns that I find to be helpful.

### Contents
- [The AnimatedValue](#the-animatedvalue)
  - [Default AnimatedValue based on props](#default-animatedvalue-based-on-props)
- [Interpolations](#interpolations)
  - [Move interpolations out of the `render()`](#move-interpolations-out-of-the-render)
  - [Move animated styles out of the `render()`](#move-animated-styles-out-of-the-render)
  - [Simplify interpolations with a helper function](#simplify-interpolations-with-a-helper-function)
  - [Interpolating with props](#interpolating-with-props)
  - [Interpolations based on measurements](#interpolations-based-on-measurements)
- [Animating](#animating)
  - [Naming and reusing animations](#naming-and-reusing-animations)
  - [Parellel, Sequential, and Staggered Animations](#parellel-sequential-and-staggered-animations)
  - [Keeping conditional animations in parellel](#keeping-conditional-animations-in-parellel)
  - [Animation helper functions](#animation-helper-functions)
  - [Declarative Animated Components](#declarative-animated-components)

## The AnimatedValue

```js
animatedValue = new Animated.Value(0)
```

Every animation you create starts with this. An Animated value can be used to compose multiple animations on a single scene. Or multiple Animated values can perform multiple transitions on a single element, like `translateY` and `opacity` in sequence.

You can use whatever style you like, but I prefer not to store Animated values within `state` when using Class components, mainly because the methods called on an Animated Value will not trigger the React Native lifecycle hooks.

For the examples in this article I will store the values in an instance variable instead.

### Default AnimatedValue based on props

If we want to create a flexible component that is animated based on props, we can default our initial values like so:

```jsx
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  constructor(props) {
    super(props)
    if (props.isVisible) {
      this.animatedValue.setValue(1)
    }
  }

  // or...

  animatedValue = new Animated.Value(this.props.isVisible ? 1 : 0)

  //...
}
```

## Interpolations

Interpolations are the next step when building an Animation. They allow you to take a range of `0 => 1` and translate that to pixel values like `0px => 100px`, degree values like `0deg => 90deg` and color values like `'tomato' => 'blanchedalmond'`.

In *most* cases, it is ideal to keep your `inputRange` to be equal to `0 => 1`. This helps with interpolating multiple values and allows for consistency when composing animations.

For example if I want to translate something 123 pixels on the x-axis, I could just animate my value from `0` to `123`, but then adding additional animated interpolations would have to use an `inputRange` of `[0, 123]`. It's better to just have a clean `inputRange` and have a more flexible `outputRange`.

```js
translateX = this.animatedValue.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 123],
})
```

An exception to this case is when interpolating values based on ScrollView's scroll value and PanResponder's X & Y values.

### Move interpolations out of the `render()`

In many basic examples, it's common to see interpolations being created within the `render()`, like so:

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  render() {
    const animatedTranslateY = this.animatedValue.interpolate({
      inputRange: [0, 1],
      outputRange: [40, 0],
    })

    const animatedOpacity = this.animatedValue.interpolate({
      inputRange: [0, 1],
      outputRange: [0, 1],
    })

    return (
      <Animated.View
        style={{
          opacity: animatedOpacity,
          transform: [
            {translateY: animatedTranslateY},
          ],
        }}
      />
    )
  }
}
```

Although this will not bog down performance too much, the `interpolate` method is being fired on every re-render. Since our interpolations are not changing, it's better to store them as instance variables:

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  animatedTranslateY = this.animatedValue.interpolate({
    inputRange: [0, 1],
    outputRange: [40, 0],
  })

  animatedOpacity = this.animatedValue.interpolate({
    inputRange: [0, 1],
    outputRange: [0, 1],
  })

  render() {
    return (
      <Animated.View
        style={[
          styles.boxStyle,
          {
            opacity: this.animatedOpacity,
            transform: [
              {translateY: this.animatedTranslateY},
            ],
          },
        ]}
      />
    )
  }
}
```

👍

### Move animated styles out of the `render()`

We can also apply this same perf-minded thinking to our styles, and create an `animatedStyle` within our class as well:

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  animatedTranslateY = this.animatedValue.interpolate({
    inputRange: [0, 1],
    outputRange: [40, 0],
  })

  animatedOpacity = this.animatedValue.interpolate({
    inputRange: [0, 1],
    outputRange: [0, 1],
  })

  animatedStyle = {
    opacity: this.animatedOpacity,
    transform: [
      {translateY: this.animatedTranslateY},
    ],
  }

  render() {
    return (
      <Animated.View
        style={[styles.boxStyle, this.animatedStyle]}
      />
    )
  }
}
```

Our render function is looking slimmer than ever.

### Simplify interpolations with a helper function

If we stick to our guns, and stick with the `0 => 1` input range rule. We can simplify our interpolation code by creating a helper function that accepts an `AnimatedValue`, a starting `outputA` value, and an ending `outputB` value. Let's call it `createSimpleInterpolation()`. We can also store this in a `utils` file.

We can store our `INPUT_RANGE` as a constant and now we can create new simple interpolations on the fly.

```js
// utils/animated.js

const INPUT_RANGE = [0, 1]

function createSimpleInterpolation(animatedValue, outputA, outputB) {
  return (
    animatedValue.interpolate({
      inputRange: INPUT_RANGE,
      outputRange: [outputA, outputB],
    })
  )
}

// MyComponent.js

class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)
  animatedTranslateY = createSimpleInterpolation(this.animatedValue, 40, 0)
  animatedOpacity = createSimpleInterpolation(this.animatedValue, 0, 1)

  //...
}
```

This is great for simple interpolations, but for more complex `inputRange`s we can just use the normal API.

### Interpolating with props

Interpolating with props is easy and is great to allow for more custom animations per scene.

A simple example is if we wanted to translate something based on a dynamic starting position.

```js
class MyComponent extends React.Component {
  //...

  animatedTranslateY = this.animatedValue.interpolate({
    inputRange: [0, 1],
    outputRange: [this.props.initialPositionY, 0],
  })

  // or using our helper

  animatedTranslateY = createSimpleInterpolation(this.animatedValue, this.props.initialPositionY, 0)

  //...
}
```

The above example allows us to pass an `initialPositionY` prop to our component, that will then animate to a position of `0`.


### Interpolations based on measurements

Interpolating based on measurements is a little bit tricky. Unlike a known property that is passed to our component, creating interpolations based on measurements requires more work.

In this example we have a `HidingNavBar` that will "hide" itself by translating the negative value of it's height, so it moves off-screen.

We measure the height by using the `onLayout` prop. And deconstructing `height` from `event.nativeEvent.layout` within our `handleLayout()` function.

```js
// a component that is negatively translated to its exact dynamic height
class HidingNavBar extends React.Component {

  state = {
    animatedStyle: null
  }

  animatedValue = new Animated.Value(0)

  animatedTranslateY = createSimpleInterpolation(this.animatedValue, -999, 0) // default

  handleLayout(event) {
    const { height } = event.nativeEvent.layout

    this.animatedTranslateY = createSimpleInterpolation(this.animatedValue, -height, 0)

    const animatedStyle = {
      transform: [{ translateY: animatedTranslateY }]
    }

    this.setState({ animatedStyle })
  }

  render() {
    return (
      <Animated.View
        onLayout={this.handleLayout}
        style={[styles.boxStyle, this.state.animatedStyle]}
      />
    )
  }
}
```
In this case, we are putting `animatedStyle` in `state` to ensure that our component re-renders with our new interpolation that uses the `height` measurement.

One thing to note here is that `onLayout` could fire multiple times. In that case we should store the initial `height` measurement and compare it against the new `height` measurement on any subsequent layout call:

```jsx
class HidingNavBar extends React.Component {
  //...
  handleLayout(event) {
    const { height } = event.nativeEvent.layout

    // if the layout height is equal to the height in our state,
    // we can return and prevent unnecessary setState calls and reassignment of variables
    if (height === this.state.height) {
      return
    }

    this.animatedTranslateY = createSimpleInterpolation(this.animatedValue, -height, 0)

    const animatedStyle = {
      transform: [{ translateY: animatedTranslateY }]
    }

    // Store height in state
    this.setState({ animatedStyle, height })
  }
  //...
}
```

## Animating

Last, but not least, we need to run the actual animations.

Animated provides three types of animations: `decay()`, `timing()` and `spring()`. These functions accept two parameters; the `AnimatedValue` we are manipulations, and an config object:

```jsx
Animated.timing(this.animatedValue, {
  toValue: 1,
  duration: 400,
  delay: 200,
  easing: Easing.linear,
  useNativeDriver: true,
})
```

These are a couple of configurable options for `timing()`, `spring()` and `decay()` have some different options.

### Naming and reusing animations

Using these timing functions we can animate any value, here is an example that triggers an animation on `componentDidMount()`

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  //...

  componentDidMount() {
    Animated.timing(this.animatedValue, {
      toValue: 1,
      duration: 400,
      useNativeDriver: true,
    }).start()
  }

  //...
}
```

Note how `.start()` is used to trigger our animation. If you don't call this, it won't animate, which means that we can store this timing animation in an instance variable.

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  //...

  fadeInAnimation = Animated.timing(this.animatedValue, {
    toValue: 1,
    duration: 400,
    useNativeDriver: true,
  })

  componentDidMount() {
    this.fadeInAnimation.start()
  }

  //...
}
```

Just like we did with interpolations and styles, we can store and reference this animation in our class, which prevents us from recreating animations unnecessarily.

Additionally we can make the inverse of this animation with a `fadeOutAnimation`

```js
class MyComponent extends React.Component {
  //...
  fadeOutAnimation = Animated.timing(this.animatedValue, {
    toValue: 0,
    duration: 400,
    useNativeDriver: true,
  })
  //...
}
```

### Parellel, Sequential, and Staggered Animations

In a lot of cases, our components will potentially have more than one `AnimatedValue`, and we will needq to animate them at different times, durations or with different easing curves. We can use `Animated.parallel`, `Animated.sequence` and `Animated.stagger` to fire these animations in conjunction.

In the example below we have `blueBoxAnimatedValue` and `orangeBoxAnimatedValue`, both values will affect a different `View`.

This is what it will look like if we want to start both animations at the same time:

```js
class MyComponent extends React.Component {

  blueBoxAnimatedValue = new Animated.Value(0)
  blueBoxAnimatedTranslateY = createSimpleInterpolation(this.blueBoxAnimatedValue 40, 0)

  orangeBoxAnimatedValue = new Animated.Value(0)
  orangeBoxAnimatedTranslateY = createSimpleInterpolation(this.orangeBoxAnimatedValue 50, 0)

  animatedBlueBoxStyle = {
    transform: [
      {translateY: this.blueBoxAnimatedTranslateY},
    ],
  }

  animatedOrangeBoxStyle = {
    transform: [
      {translateY: this.orangeBoxAnimatedTranslateY},
    ],
  }

  blueBoxAnimation = Animated.timing(this.blueBoxAnimatedValue, {
    toValue: 1,
    duration: 800,
    useNativeDriver: true,
  })

  orangeBoxAnimation = Animated.timing(this.orangeBoxAnimatedValue, {
    toValue: 1,
    duration: 400,
    useNativeDriver: true,
  })

  componentDidMount() {
    this.animateBoxes()
  }

  animateBoxes() {
    Animated.parallel([
      this.blueBoxAnimation,
      this.orangeBoxAnimation,
    ]).start()
  }

  render() {
    return (
      <View>
        <Animated.View
          style={[styles.blueBoxStyle, this.animatedBlueBoxStyle]}
        />
        <Animated.View
          style={[styles.orangeBoxStyle, this.animatedOrangeBoxStyle]}
        />
      </View>
    )
  }
}
```

We are able to pass each of our animated timing functions as an array into `Animated.parallel`.

We can also do this with `sequence()` and `stagger()`:

```jsx
  animateBoxes() {
    Animated.sequence([
      this.blueBoxAnimation,
      this.orangeBoxAnimation,
    ]).start()
  }

  // or...

  animateBoxes() {
    Animated.stagger(100, [
      this.blueBoxAnimation,
      this.orangeBoxAnimation,
    ]).start()
  }
```

### Keeping conditional animations in parellel

Great, now imagine we're making a component that will only animate the `orangeBox` if the prop of `shouldAnimateOrange` has a value of `true`.

This *can* be achieved by doing this:

```jsx
  animateBoxes() {
    this.blueBoxAnimation.start()

    if (this.props.shouldAnimateOrange) {
      this.orangeBoxAnimation.start()
    }
  }
```

But if we want to keep the animations in parallel we can create an array of `parallelAnimations` and conditionally push whatever animations are needed.

```jsx
  animateBoxes() {
    const parallelAnimations = [this.blueBoxAnimation]

    if (this.props.shouldAnimateOrange) {
      parallelAnimations.push(this.orangeBoxAnimation)
    }

    Animated.parallel(parallelAnimations).start()
  }
```

Again, this can also be applied to `sequence()` and `stagger()`

### Animation helper functions

Similar to the interpolation helper function, you can have a set of `Animated.timing` or `.spring` functions that will help you keep consistency in your animations.

Let's set up two functions that we will hypothetically use throughout the entirety our app:

```jsx
function createTimingAnimation(animatedValue, toValue) {
  return Animated.timing(animatedValue, {
    toValue,
    duration: 600,
    easing: Easing.inOut(Easing.quad),
    useNativeDriver: true,
  })
}

function createSpringAnimation(animatedValue, toValue) {
  return Animated.spring(animatedValue, {
    toValue,
    tension: 60,
    friction: 10,
    easing: Easing.out(Easing.quad),
  })
}
```

We have one `createTimingAnimation` function that will always produce a `Animated.timing()` animation with a consistent duration of `600ms` and will always have an easing curve of `easeInOutQuad`. We also have a function that will create an `Animated.spring` animation.

We can use these helpers like so:

```jsx
class MyComponent extends React.Component {
  animatedValue = new AnimatedValue(0)
  otherAnimatedValue = new AnimatedValue(0)

  fadeInAnimation = createTimingAnimation(this.animatedValue, 1)
  fadeOutAnimation = createTimingAnimation(this.animatedValue, 0)

  springyAnimation = createSpringAnimation(this.otherAnimatedValue, 0)
  anotherSpringyAnimation = createSpringAnimation(this.otherAnimatedValue, 500)

  animate() {
    this.anotherSpringyAnimation.start()
  }
  //...
}
```

### Declarative Animated Components

Now let's create a declaratively animated component. One that will animate based on the properties that are passed to it.

The key is to fire our animations on `componentWillUpdate()`

In this example we have a `prop` of `isVisible`, which will determine whether or not we should show or hide our component by translating it on and off the screen appropriately.

First let's setup our `AnimatedValue` with a default value that is determined by `this.props.isVisible`

In this case an animated value of `0` is hidden, and `1` is visible.

```jsx
class HidingNavBar extends React.Component {

  animatedValue = new Animated.Value(this.props.isVisible ? 1 : 0)

}
```

Then we can add in our `animatedTranslateY` and our `animatedStyle`, and also create our show and hide animations using our helper.

```jsx
class HidingNavBar extends React.Component {

  animatedValue = new Animated.Value(this.props.isVisible ? 1 : 0)

  animatedTranslateY = createSimpleInterpolation(this.animatedValue, -this.props.height, 0)

  animatedStyle = {
    transform: [{translateY: this.animatedTranslateY}],
  }

  showAnimation = createTimingAnimation(this.animatedValue, 1)
  hideAnimation = createTimingAnimation(this.animatedValue, 0)

  render() {
    return (
      <Animated.View
        style={[styles.boxStyle, this.animatedStyle]}
      />
    )
  }
}
```

Within `componentWillUpdate`, we will need to make sure that the current `this.props.isVisible` does not equal the `nextProps.isVisible`

```jsx
  componentWillUpdate(nextProps) {
    if (nextProps.isVisible !== this.props.isVisible) {
      // perform animation...
    }
  }
```

Then we can create another function that will determine whether we will fire a `showAnimation` or a `hideAnimation` based on the value of `nextProps.isVisible`

```js
  componentWillUpdate(nextProps) {
    if (nextProps.isVisible !== this.props.isVisible) {
      this.animateVisibility(nextProps.isVisible)
    }
  }

  animateVisibility(isVisible) {
    if (isVisible) {
      this.showAnimation.start()
    }
    else {
      this.hideAnimation.start()
    }
  }
```

Now whenever our component's `isVisible` prop is updated, our component should animate appropriately.

Here is all the code for our `HidingNavBar` (sans styles):

```jsx
class HidingNavBar extends React.Component {

  animatedValue = new Animated.Value(this.props.isVisible ? 1 : 0)

  animatedTranslateY = createSimpleInterpolation(this.animatedValue, -this.props.height, 0)

  animatedStyle = {
    transform: [{translateY: this.animatedTranslateY}],
  }

  showAnimation = createTimingAnimation(this.animatedValue, 1)
  hideAnimation = createTimingAnimation(this.animatedValue, 0)

  componentWillUpdate(nextProps) {
    if (nextProps.isVisible !== this.props.isVisible) {
      this.animateVisibility(nextProps.isVisible)
    }
  }

  animateVisibility(isVisible) {
    if (isVisible) {
      this.showAnimation.start()
    }
    else {
      this.hideAnimation.start()
    }
  }

  render() {
    return (
      <Animated.View
        style={[styles.boxStyle, this.animatedStyle]}
      />
    )
  }
}
```

And now we can declaratively animate our `HiddenNavBar` component inside of our parent component:

```jsx
class Parent extends React.Component {
  render() {
    return(
      <View>
        <HiddenNavBar
          isVisible={this.state.isNavBarVisible}
        />
      </View>
    )
  }
}
```