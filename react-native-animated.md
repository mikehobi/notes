# Tips for React Native Animated


## Interpolations

### Move Interpolations out of the `render()`

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

to...

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


### Store styles into Class instance variables


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

### Simplify Interpolations w/ a Helper

```js
// A function that returns an interpolation assuming an `inputRange` of 0 to 1
function createSimpleInterpolation(animatedValue, outputA, outputB) {
  const inputRange = [0, 1]
  const outputRange = [outputA, outputB]

  return (
    animatedValue.interpolate({ inputRange, outputRange })
  )
}

class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)
  animatedTranslateY = createSimpleInterpolation(this.animatedValue, 40, 0)
  animatedOpacity = createSimpleInterpolation(this.animatedValue, 0, 1)

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

### interpolations with dynamic values?

How can you create interpolations based on changing values if you're not creating them in the render?

```js

class MyComponent extends React.Component {
  // ...

  myInterpolation = this.animatedValue.interpolate({
    inputRange: [0, 1],
    outputRange: [this.props.offsetY, 0],
  })

  // or
  this.myInterpolation = createSimpleInterpolation(this.animatedValue, this.props.offsetY, 0)

  // ...
}

```

### interpolations based on measurements

```js
// a component that is negatively translated to its exact dynamic height
class HidingNavBar extends React.Component {

  state = {
    animatedStyle: null
  }

  animatedValue = new Animated.Value(0)

  animatedTranslateY = createSimpleInterpolation(this.animatedValue, -999, 0) // default value... not really needed...

  handleLayout(event) {
    const { height } = event.nativeEvent.layout

    this.animatedTranslateY = createSimpleInterpolation(this.animatedValue, -height, 0)

    const animatedStyle = {
      transform: [{ translateY: animatedTranslateY }]
    }

    this.setState({ animatedStyle })
  }

  render() {
    <Animated.View
      onLayout={this.handleLayout}
      style={[styles.boxStyle, this.state.animatedStyle]}
    />
  }
}
```

In this example we have to wait for our component to be mounted, and have it's `onLayout` callback be fired before we can determine our interpolation. Therefore we *need* to store our style within our state to insure a re-render. Otherwise, our interpolation would still be our default of `[-999, 0]`

### default Animated value based on props

```jsx
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)

  constructor(props) {
    super(props)
    if (props.isVisible) {
      this.animatedValue.setValue(1)
    }
  }

}
```


## Animating Values

### Naming and reusing animations.

from...

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)
  animatedTranslateY = createSimpleInterpolation(this.animatedValue, 40, 0)
  animatedOpacity = createSimpleInterpolation(this.animatedValue, 0, 1)

  animatedStyle = {
    opacity: this.animatedOpacity,
    transform: [
      {translateY: this.animatedTranslateY},
    ],
  }

  componentDidMount() {
    Animated.timing(this.animatedValue, {
      toValue: 1,
      duration: 400,
      useNativeDriver: true,
    }).start()
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
to...

```js
class MyComponent extends React.Component {

  animatedValue = new Animated.Value(0)
  animatedTranslateY = createSimpleInterpolation(this.animatedValue, 40, 0)
  animatedOpacity = createSimpleInterpolation(this.animatedValue, 0, 1)

  animatedStyle = {
    opacity: this.animatedOpacity,
    transform: [
      {translateY: this.animatedTranslateY},
    ],
  }

  fadeInAnimation = Animated.timing(this.animatedValue, {
    toValue: 1,
    duration: 400,
    useNativeDriver: true,
  })

  componentDidMount() {
    this.fadeInAnimation.start()
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

additionally...

```js
class MyComponent extends React.Component {
  // ...
  fadeOutAnimation = Animated.timing(this.animatedValue, {
    toValue: 0,
    duration: 400,
    useNativeDriver: true,
  })
  // ...
}
```

### Parellel animations...

Animating multiple animated values...

two boxes, different animated values... parallel animations

```js
class MyComponent extends React.Component {

  animatedBlueBoxValue = new Animated.Value(0)
  animatedBlueTranslateY = createSimpleInterpolation(this.animatedBlueBoxValue 40, 0)

  animatedOrangeBoxValue = new Animated.Value(0)
  animatedOrangeTranslateY = createSimpleInterpolation(this.animatedOrangeBoxValue 50, 0)

  animatedBlueBoxStyle = {
    transform: [
      {translateY: this.animatedBlueTranslateY},
    ],
  }

  animatedOrangeBoxStyle = {
    transform: [
      {translateY: this.animatedOrangeTranslateY},
    ],
  }

  translateBlueAnimation = Animated.timing(this.animatedBlueBoxValue, {
    toValue: 1,
    duration: 800,
    useNativeDriver: true,
  })

  translateOrangeAnimation = Animated.timing(this.animatedOrangeBoxValue, {
    toValue: 1,
    duration: 400,
    useNativeDriver: true,
  })

  componentDidMount() {
    this.animateBoxes()
  }

  animateBoxes() {
    Animated.parallel([
      this.translateBlueAnimation,
      this.translateOrangeAnimation,
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

keeping parallel-zation with boolean animations...

instead of...

```jsx
  // no!
  animateBoxes() {
    this.translateBlueAnimation.start()
    if (this.props.shouldAnimateOrange) {
      this.translateOrangeAnimation.start()
    }
  }
```

keep animations in parallel

```jsx
  // yes!
  animateBoxes() {
    const parallelAnimations = [this.translateBlueAnimation]

    if (this.props.shouldAnimateOrange) {
      parallelAnimations.push(this.translateOrangeAnimation)
    }

    Animated.parallel(parallelAnimations).start()
  }
```

### Declarative Animations within components

Hidden navbar w/ fixed height

initialize within `constructor()`

```jsx
class HidingNavBar extends React.Component {

  animatedValue = new Animated.Value(0)
  animatedTranslateY = null

  constructor(props) {
    super(props)

    this.animatedValue.setValue(props.isVisible ? 1 : 0)

    this.animatedTranslateY = createSimpleInterpolation(this.animatedValue, -props.height, 0)

    const animatedStyle = {
      transform: [{translateY: this.animatedTranslateY}],
    }

    this.state = {
      animatedStyle,
    }
  }

  showAnimation = Animated.timing(this.animatedValue, {
    toValue: 1,
    duration: 400,
    useNativeDriver: true,
  })

  hideAnimation = Animated.timing(this.animatedValue, {
    toValue: 0,
    duration: 400,
    useNativeDriver: true,
  })

  componentWillUpdate(nextProps) {
    if (nextProps.isVisible !== this.props.isVisible) {
      this.animateVisibility(nextProps.isVisible)
    }
  }

  animateVisibility(isVisible) {
    if (isVisible) {
      this.hideAnimation.start()
    }
    else {
      this.showAnimation.start()
    }
  }

  render() {
    <Animated.View
      onLayout={this.handleLayout}
      style={[styles.boxStyle, this.state.animatedStyle]}
    />
  }
}
```

Now we can declaratively hide or show our `HiddenNavBar` component:

```jsx
class Parent extends React.Component {
  render() {
    return(
      <HiddenNavBar
        // hide it...
        isVisible={false}
        // or control w/ state
        isVisible={this.state.isNavBarVisible}
      />
    )
  }
}
```

### animation helper functions?

Similar to an interpolation helper, you can have set `Animated.timing` or `.spring` functions to keep consistency within in your app.

Not sure if this is helpful. Could create bad/confusing patterns...

```jsx
function createAnimation(animatedValue, toValue) {
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


class MyComponent extends React.Component {
  animatedValue = new AnimatedValue(0)
  otherAnimatedValue = new AnimatedValue(0)

  fadeInAnimation = createAnimation(this.animatedValue, 1)
  fadeOutAnimation = createAnimation(this.animatedValue, 0)

  springyAnimation = createSpringAnimation(this.otherAnimatedValue, 0)
  anotherSpringyAnimation = createSpringAnimation(this.otherAnimatedValue, 500)

  animate() {
    this.anotherOtherAnimation.start()
  }
}
```