# Creating a Custom Animated TabBar fro `react-native-tab-view`

Let's start with a basic implementation of `TabViewAnimated`. And break this down piece by piece.

```jsx
import React from 'react'
import {
  View,
  Text,
  StyleSheet,
} from 'react-native'
import { bind } from 'decko'
import { TabViewAnimated } from 'react-native-tab-view'

export default class MyTabView extends React.Component {
  state = {
    index: 0,
    routes: [
      { tabIndex: 0, key: '0', label: `Tab #1` },
      { tabIndex: 1, key: '1', label: `Tab #2` },
      { tabIndex: 2, key: '2', label: `Tab #3` },
      { tabIndex: 3, key: '3', label: `Tab #4` },
    ],
  }

  @bind
  handleChangeTab(index) {
    this.setState({ index })
  }

  @bind
  renderScene(props) {
    return (
      <View style={styles.scene}>
        <Text>{`Hi I'm ${props.route.label}`}</Text>
      </View>
    )
  }

  render() {
    return (
      <TabViewAnimated
        navigationState={this.state}
        renderScene={this.renderScene}
        // renderHeader={this.renderHeader}
        onRequestChangeTab={this.handleChangeTab}
      />
    )
  }
}

const styles = StyleSheet.create({
  scene: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
})

```

First the imports...

```jsx
import React from 'react'
import {
  View,
  Text,
  StyleSheet,
} from 'react-native'
import { bind } from 'decko'
import { TabViewAnimated } from 'react-native-tab-view'
```

Most of these should be obvious, but it's worth noting that I am using `bind` from the `decko` package (you can use your preferred method to bind your functions).

And of course we are importing `TabViewAnimated` from `react-native-tab-view`.

...

In our component's state we are defining our default route `index` and we also have our route array:

```js

  state = {
    index: 0,
    routes: [
      { key: '1', label: 'Tab #1' },
      { key: '2', label: 'Tab #2' },
      { key: '3', label: 'Tab #3' },
      { key: '4', label: 'Tab #4' },
    ],
  }

```

`key` is the only thing required by `TabViewAnimated` to work properly, a `label` that will be used for presentation purposes, and a `tabIndex`.

```js
  @bind
  handleChangeTab(index) {
    this.setState({ index })
  }

  @bind
  renderScene(props) {
    return (
      <View style={styles.scene}>
        <Text>{`Hi I'm ${props.route.label}`}</Text>
      </View>
    )
  }

  render() {
    return (
      <TabViewAnimated
        navigationState={this.state}
        renderScene={this.renderScene}
        // renderHeader={this.renderHeader}
        onRequestChangeTab={this.handleChangeTab}
      />
    )
  }
```

And then we have our `handleChangeTab` function and our `renderScene` funciton. Both our pased down to `TabViewAnimated` along with `this.state` as `navigationState`.

So this is a very basic implementation that will give you 4 tabs that you can swipe between.



Let's create a very simple `renderHeader` function, and then we will build on top of it.

We going to create 2 additional components to go along with our `MyTabView` component: `TabHeader` and `TabLink`.

`TabHeader` will be a component that maps over our `routes` array and creates a `TabLink` for each one.

In our parent we will create our `renderHeader` function and pass down a few necessary props. Our `routes`, the current `index`, and `jumpToIndex` a method provided by `TabViewAnimated` that allows us to jump to tabs.

```jsx
  //...

  @bind
  renderHeader(props) {
    return (
      <TabHeader
        routes={props.navigationState.routes}
        index={props.navigationState.index}
        jumpToIndex={props.jumpToIndex}
      />
    )
  }

  //...

  render() {
    return (
      <TabViewAnimated
        navigationState={this.state}
        renderScene={this.renderScene}
        // Pass our function down to `TabViewAnimated`
        renderHeader={this.renderHeader}
        onRequestChangeTab={this.handleChangeTab}
      />
    )
  }
```

Now let's make `TabHeader`

```jsx
class TabHeader extends React.Component {
  @bind
  handleTabPress(index) {
    this.props.jumpToIndex(index)
  }

  render() {
    return (
      <View
        style={styles.header}
      >
        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              onPress={this.handleTabPress}
            />
          )
        )}
      </View>
    )
  }
}
```

If we look at our `render`, we have a styled `View`, and with in that we have a map function that is returning a `TabLink` for every route.

We our using our `route.key` as the `key` prop.

We our passing down `route.tabIndex` as `index` to each `TabLink` (We'll see why later...)

We have our `label` and we our passing down a boolean that compares the `tabIndex` to the `this.props.index`, which is the current index from `navigationState`. We will use this to highlight which tab link is active.

And we also are passing down an `onPress` which expects an `index` that will call `this.props.jumpToIndex`. You can probably see where this is going. Let's checkout `TabLink`


```jsx
class TabLink extends React.Component {

  @bind
  handlePress() {
    this.props.onPress(this.props.index)
  }

  render() {
    return (
      <TouchableOpacity
        onPress={this.handlePress}
      >
        <View
          style={styles.tabContainer}
        >
          <Text
            style={{
              color: this.props.isActive ? '#000' : '#CCC',
            }}
          >
            {this.props.label}
          </Text>
        </View>
      </TouchableOpacity>
    )
  }
}
```

We have a `TouchableOpacity` with an `onPress` event that will call `this.props.onPress` and pass `this.props.index` up to `TabHeader`.

We also have a styled `View` and within that we have a `Text` component with a dynamic style that will change `color` depending on `this.props.isActive`.

So if our `TabLink` is active, it will have a text color of `#000`.

Here is the entire component.

```jsx

import React from 'react'
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
} from 'react-native'
import { bind } from 'decko'
import { TabViewAnimated } from 'react-native-tab-view'

export default class MyTabView extends React.Component {

  state = {
    index: 0,
    routes: [
      { tabIndex: 0, key: '0', label: `Tab #1` },
      { tabIndex: 1, key: '1', label: `Tab #2` },
      { tabIndex: 2, key: '2', label: `Tab #3` },
      { tabIndex: 3, key: '3', label: `Tab #4` },
    ],
  }

  @bind
  renderHeader(props) {
    return (
      <TabHeader
        routes={props.navigationState.routes}
        index={props.navigationState.index}
        jumpToIndex={props.jumpToIndex}
      />
    )
  }

  @bind
  renderScene(props) {
    return (
      <View style={styles.scene} >
        <Text>{`Hi I'm ${props.route.label}`}</Text>
      </View>
    )
  }

  @bind
  handleChangeTab(index) {
    this.setState({ index })
  }

  render() {
    return (
      <TabViewAnimated
        navigationState={this.state}
        renderHeader={this.renderHeader}
        renderScene={this.renderScene}
        onRequestChangeTab={this.handleChangeTab}
      />
    )
  }
}

class TabHeader extends React.Component {
  @bind
  handleTabPress(index) {
    this.props.jumpToIndex(index)
  }

  render() {
    return (
      <View
        style={styles.header}
      >
        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              onPress={this.handleTabPress}
            />
          )
        )}
      </View>
    )
  }
}

class TabLink extends React.Component {

  @bind
  handlePress() {
    this.props.onPress(this.props.index)
  }

  render() {
    return (
      <TouchableOpacity
        onPress={this.handlePress}
      >
        <View
          style={styles.tabContainer}
        >
          <Text
            style={{
              color: this.props.isActive ? '#000' : '#CCC',
            }}
          >
            {this.props.label}
          </Text>
        </View>
      </TouchableOpacity>
    )
  }
}


const styles = StyleSheet.create({
  scene: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  header: {
    marginTop: 20,
    height: 60,
    flexDirection: 'row',
    alignItems: 'center',
  },
  tabContainer: {
    marginHorizontal: 8,
  },
})
```

This is pretty sweet. But as you can see, the `color` style doesn't update until the the state change of `index` has completed. There is a delay because it is waiting for the `ScrollView` to end its momentum.


We can make this a lot smoother, instead of relying on `navigationState.index` we can actually set a listener on the current `position` of the Tab View. Luckily, `TabViewAnimated` passes this as a prop in our `renderHeader` function.

```jsx
export default class MyTabView extends React.Component {
  //...
  @bind
  renderHeader(props) {
    return (
      <TabHeader
        routes={props.navigationState.routes}
        index={props.navigationState.index}
        jumpToIndex={props.jumpToIndex}
        // Pass down `props.subscribe`
        subscribe={props.subscribe}
      />
    )
  }
  //...
}

class TabHeader extends React.Component {
  //...
  componentDidMount() {
    this.props.subscribe('position', this.handlePositionChange)
  }

  @bind
  handlePositionChange(position) {
    console.log(position)
  }
  //...
}
```

Here is the basic setup to get the current `position`

If we check out our console, we'll see some action when we swipe between the tabs.


So now let's set it up to be `Animated.Value`.

We're going to use this `Animated.Value` to create an interpolation for each `TabLink` so that when the `position` matches the index of a `TabLink`, the color should fade in accordingly.

```jsx
class TabHeader extends React.Component {
  //...
  positionAnimatedValue = new Animated.Value(this.props.index) // initialize with the default/current index

  componentDidMount() {
    this.props.subscribe('position', this.handlePositionChange)
  }

  @bind
  handlePositionChange(position) {
    // Here we our explicitly calling `setValue` on our `positionAnimatedValue` to our `position`
    // Ideally we would be able to use an `Animated.event` but that is not currently supported with `TabViewAnimated`
    this.positionAnimatedValue.setValue(position)
  }
  //...
}
```

Now that we have `positionAnimatedValue` we can begin setting up our interpolations.

In `TabHeader` we are going to call a function called `createAnimatedTabInterpolations` that will accept `routes` as a parameter.

```jsx
class TabHeader extends React.Component {
  //...
  constructor(props) {
    super(props)
    this.createAnimatedTabInterpolations(props.routes)
  }
  //...
}
```

This function will then map over each `route` and return an array: `tabTextColorInterpolations`, that contains a text color interpolation for each `tabIndex`.

```jsx
class TabHeader extends React.Component {
  //...
  tabTextColorInterpolations = []

  @bind
  createAnimatedTabInterpolations(routes) {
    this.tabTextColorInterpolations = routes.map(
      (route) => {
        // return a text color interpolation into array
        return this.positionAnimatedValue.interpolate({
          inputRange: [route.tabIndex - 1, route.tabIndex, route.tabIndex + 1],
          outputRange: ['#CCC', '#000', '#CCC'],
          extrapolate: 'clamp',
        })
      }
    )
  }
  //...
}
```

Awesome. A neat trick here is that we only need to interpolate between the tab's siblings and the current tab: `[route.tabIndex - 1, route.tabIndex, route.tabIndex + 1]` or `[previousTab, currentTab, nextTab]`

Now all we need to do is pass down the appropriate interpolation to each `TabLink`

```jsx
class TabHeader extends React.Component {
  //...
  render() {
    return (
      <View
        style={styles.header}
      >
        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              // textColor is a new prop that will use the right interpolation for the tabIndex provided.
              textColor={this.tabTextColorInterpolations[route.tabIndex]}
              onPress={this.handleTabPress}
            />
          )
        )}
      </View>
    )
  }
  //...
}
```

And in our `TabLink` component:

```jsx

class TabLink extends React.Component {
  //...
  render() {
    return (
      <TouchableOpacity
        onPress={this.handlePress}
      >
        <View
          style={styles.tabContainer}
        >
          <Animated.Text
            // here we use the interpolation that we pass down
            style={{ color: this.props.textColor }}
          >
            {this.props.label}
          </Animated.Text>
        </View>
      </TouchableOpacity>
    )
  }
  //...
}
```

So now we get a smooth interpolation of text color based on the `position` value.

Again here is the code in it's entirety:

```jsx


import React from 'react'
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Animated,
} from 'react-native'
import { bind } from 'decko'
import { TabViewAnimated } from 'react-native-tab-view'

export default class MyTabView extends React.Component {

  state = {
    index: 0,
    routes: [
      { tabIndex: 0, key: '0', label: `Tab #1` },
      { tabIndex: 1, key: '1', label: `Tab #2` },
      { tabIndex: 2, key: '2', label: `Tab #3` },
      { tabIndex: 3, key: '3', label: `Tab #4` },
    ],
  }

  @bind
  renderHeader(props) {
    return (
      <MyTabViewHeader
        routes={props.navigationState.routes}
        index={props.navigationState.index}
        jumpToIndex={props.jumpToIndex}
        subscribe={props.subscribe}
      />
    )
  }

  @bind
  renderScene(props) {
    return (
      <View
        style={styles.scene}
      >
        <Text>{`Hi I'm ${props.route.label}`}</Text>
      </View>
    )
  }

  @bind
  handleChangeTab(index) {
    this.setState({ index })
  }

  render() {
    return (
      <TabViewAnimated
        navigationState={this.state}
        renderHeader={this.renderHeader}
        renderScene={this.renderScene}
        onRequestChangeTab={this.handleChangeTab}
      />
    )
  }
}

class TabHeader extends React.Component {

  constructor(props) {
    super(props)
    this.createAnimatedTabInterpolations(props.routes)
  }

  componentDidMount() {
    this.props.subscribe('position', this.handlePositionChange)
  }

  positionAnimatedValue = new Animated.Value(this.props.index)

  @bind
  createAnimatedTabInterpolations(routes) {
    this.tabTextColorInterpolations = routes.map(
      (route) => {
        return this.positionAnimatedValue.interpolate({
          inputRange: [route.tabIndex - 1, route.tabIndex, route.tabIndex + 1],
          outputRange: ['#CCC', '#000', '#CCC'],
          extrapolate: 'clamp',
        })
      }
    )
  }

  @bind
  handlePositionChange(position) {
    this.positionAnimatedValue.setValue(position)
  }

  @bind
  handleTabPress(index) {
    this.props.jumpToIndex(index)
  }

  render() {
    return (
      <View
        style={styles.header}
      >
        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              textColor={this.tabTextColorInterpolations[route.tabIndex]}
              onPress={this.handleTabPress}
            />
          )
        )}
      </View>
    )
  }
}

class TabLink extends React.Component {

  @bind
  handlePress() {
    this.props.onPress(this.props.index)
  }

  render() {
    return (
      <TouchableOpacity
        onPress={this.handlePress}
      >
        <View
          style={styles.tabContainer}
        >
          <Animated.Text
            style={{ color: this.props.textColor }}
          >
            {this.props.label}
          </Animated.Text>
        </View>
      </TouchableOpacity>
    )
  }
}

const styles = StyleSheet.create({
  scene: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  header: {
    marginTop: 20,
    height: 60,
    flexDirection: 'row',
    alignItems: 'center',
  },
  tabContainer: {
    marginHorizontal: 8,
  },
})
```

Snack link: [...]

Awesome. Now let's take it another step further and build an indicator bar. But rather than just a set-width indicator bar, let's add one that changes sizes based on our `TabLink` widths.

In order to achieve this affect we are going to measure the `width` and `x` position of each `TabLink`. We will use these measurements to create an interpolation on our indicator. So when our `position` value changes, the `x` position of our indicator will move to the appropriate `TabLink` and the `width` will be the same `width` of the current `TabLink`.

For the sake of this example, let's change our route labels to varying strings sizes.

```jsx
export default class MyTabView extends React.Component {
  //...
  state = {
    index: 0,
    routes: [
      { tabIndex: 0, key: '0', label: `Tab #1` },
      { tabIndex: 1, key: '1', label: `Wide Tab #2` },
      { tabIndex: 2, key: '2', label: `#3` },
      { tabIndex: 3, key: '3', label: `Really Wide Tab #4` },
    ],
  }
  //...
}
```

Before we get into the measurements, let's add an `Animated.View` that will act as our indicator. You can give it any style you want, in this example it will be full-height and rendered behind the text labels.

```jsx
class TabHeader extends React.Component {
  //...
  state = {
    tabIndicatorAnimatedStyle: null,
  }
  //...
  render() {
    return (
      <View
        style={styles.header}
      >

        <Animated.View
          style={[
            styles.tabIndicator,
            this.state.tabIndicatorAnimatedStyle,
          ]}
        />

        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              textColor={this.tabTextColorInterpolations[route.tabIndex]}
              onPress={this.handleTabPress}
            />
          )
        )}

      </View>
    )
  }
}

const styles = StyleSheet.create({
  //...
  tabIndicator: {
    position: 'absolute',
    left: 0,
    top: 0,
    height: '100%',
    backgroundColor: 'tomato',
  },
  //...
})
```

Sweet. Notice `tabIndicatorAnimatedStyle` in our state. After we finish measuring the tabs, we will store the `translateX` and `width` interpolations here.

```jsx
class TabHeader extends React.Component {
  //...
  render() {
    return (
      <View
        style={styles.header}
      >

        <Animated.View
          style={[
            styles.tabIndicator,
            this.state.tabIndicatorAnimatedStyle,
          ]}
        />

        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              textColor={this.tabTextColorInterpolations[route.tabIndex]}
              onPress={this.handleTabPress}
              // Pass `onLayout` prop:
              onLayout={this.handleTabLinkLayout}
            />
          )
        )}

      </View>
    )
  }
}


class TabLink extends React.PureComponent {
  @bind
  handleLayout(event) {
    // Call onLayout prop with event and also the tab's index.
    this.props.onLayout(event, this.props.index)
  }
  //...
  render() {
    return (
      <TouchableOpacity
        onPress={this.handlePress}
        onLayout={this.handleLayout}
      >
        <View
          style={styles.tabContainer}
        >
          <Animated.Text
            style={{ color: this.props.textColor }}
          >
            {this.props.label}
          </Animated.Text>
        </View>
      </TouchableOpacity>
    )
  }
}
```

In `TabHeader`, our `handleTabLinkLayout` event will look like this:

```jsx
class TabHeader extends React.Component {
  //...
  this.tabMeasurements = {}

  @bind
  handleTabLinkLayout(event, tabIndex) {
    const { x, width } = event.nativeEvent.layout

    // Store tab measurements in the tabMeasurements object with a key of it's index
    Object.assign(this.tabMeasurements, {
      [tabIndex]: {
        x,
        width,
      },
    })

    // cCmpare the length of our object to the length of our routes
    const areTabsMeasured = Object.keys(this.tabMeasurements).length === this.props.routes.length

    if (areTabsMeasured && !this.state.tabIndicatorAnimatedStyle) {
      this.setTabIndicatorInterpolations()
    }
  }
  //...
}
```

Each `TabLink` will fire `onLayout` at different times, we take each measurement and store it in an Object called `tabMeasurements`.

On each measurement, we can then compare the length of `tabMeasurements` against the length of `this.props.routes`. When these are equal, we know that we have measurements for each `TabLink` and then we can fire `this.setTabIndicatorInterpolations`.

Our `setTabIndicatorInterpolations` will then take an array of our tab indices and map that to their respective widths and x positions.

```jsx
class TabHeader extends React.Component {
  //...
  @bind
  setTabIndicatorInterpolations() {
    // An Array of tab indices, ex. [0, 1, 2, ...]
    const tabIndices = Object.keys(this.tabMeasurements).map((key) => {
      return Number(key)
    })

    // Array of tab x positions, ex. [0, 20, 40, ...]
    const tabPositions = tabIndices.map((tabIndex) => {
      return this.tabMeasurements[tabIndex].x
    })

    // Array of tab widths, ex. [50, 50, 100, ...]
    const tabWidths = tabIndices.map((tabIndex) => {
      return this.tabMeasurements[tabIndex].width
    })

    // Interpolate the `positionAnimatedValue` to each tabs x value  [0, 1, 2, ...] => [0, 20, 40, ...]
    const tabIndicatorTranslateX = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabPositions,
    })

    // Interpolate the `positionAnimatedValue` to each tabs width value  [0, 1, 2, ...] => [50, 50, 100, ...]
    const tabIndicatorWidth = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabWidths,
    })

    // Create our Animated style
    const tabIndicatorAnimatedStyle = {
      width: tabIndicatorWidth,
      transform: [{ translateX: tabIndicatorTranslateX }],
    }

    // Set state to rerender our tab indicator with new styles
    this.setState({ tabIndicatorAnimatedStyle })
  }
  //...
}
```

And that's it!

Now when we run our app. We'll see our indicator bar follow the active tabs based on the `position` value.

```jsx

import React from 'react'
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Animated,
} from 'react-native'
import { bind } from 'decko'
import { TabViewAnimated } from 'react-native-tab-view'

export default class MyTabView extends React.Component {

  state = {
    index: 0,
    routes: [
      { tabIndex: 0, key: '0', label: `Tab #1` },
      { tabIndex: 1, key: '1', label: `Wide Tab #2` },
      { tabIndex: 2, key: '2', label: `#3` },
      { tabIndex: 3, key: '3', label: `Really Wide Tab #4` },
    ],
  }

  @bind
  renderHeader(props) {
    return (
      <TabHeader
        routes={props.navigationState.routes}
        index={props.navigationState.index}
        jumpToIndex={props.jumpToIndex}
        subscribe={props.subscribe}
      />
    )
  }

  @bind
  renderScene(props) {
    return (
      <View
        style={styles.scene}
      >
        <Text>{`Hi I'm ${props.route.label}`}</Text>
      </View>
    )
  }

  @bind
  handleChangeTab(index) {
    this.setState({ index })
  }

  render() {
    return (
      <TabViewAnimated
        navigationState={this.state}
        renderHeader={this.renderHeader}
        renderScene={this.renderScene}
        onRequestChangeTab={this.handleChangeTab}
      />
    )
  }
}

class TabHeader extends React.Component {

  constructor(props) {
    super(props)
    this.createAnimatedTabInterpolations(props.routes)
  }

  state = {
    tabIndicatorAnimatedStyle: null,
  }

  componentDidMount() {
    this.props.subscribe('position', this.handlePositionChange)
  }

  positionAnimatedValue = new Animated.Value(0)
  tabMeasurements = {}

  @bind
  createAnimatedTabInterpolations(routes) {
    this.tabTextColorInterpolations = routes.map(
      (route) => {
        return this.positionAnimatedValue.interpolate({
          inputRange: [route.tabIndex - 1, route.tabIndex, route.tabIndex + 1],
          outputRange: ['#CCC', '#000', '#CCC'],
          extrapolate: 'clamp',
        })
      }
    )
  }

  @bind
  handlePositionChange(position) {
    this.positionAnimatedValue.setValue(position)
  }

  @bind
  handleTabPress(index) {
    this.props.jumpToIndex(index)
  }

  @bind
  handleTabLinkLayout(event, tabIndex) {
    const { x, width } = event.nativeEvent.layout

    Object.assign(this.tabMeasurements, {
      [tabIndex]: {
        x,
        width,
      },
    })

    const areTabsMeasured = Object.keys(this.tabMeasurements).length === this.props.routes.length

    if (areTabsMeasured && !this.state.tabIndicatorAnimatedStyle) {
      this.setTabIndicatorInterpolations()
    }
  }

  @bind
  setTabIndicatorInterpolations() {
    const tabIndices = Object.keys(this.tabMeasurements).map((key) => {
      return Number(key)
    })

    const tabPositions = tabIndices.map((tabIndex) => {
      return this.tabMeasurements[tabIndex].x
    })

    const tabWidths = tabIndices.map((tabIndex) => {
      return this.tabMeasurements[tabIndex].width
    })

    const tabIndicatorTranslateX = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabPositions,
    })

    const tabIndicatorWidth = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabWidths,
    })

    const tabIndicatorAnimatedStyle = {
      width: tabIndicatorWidth,
      transform: [{ translateX: tabIndicatorTranslateX }],
    }

    this.setState({ tabIndicatorAnimatedStyle })
  }

  render() {
    return (
      <View
        style={styles.header}
      >

        <Animated.View
          style={[
            styles.tabIndicator,
            this.state.tabIndicatorAnimatedStyle,
          ]}
        />

        {this.props.routes && this.props.routes.map(
          (route) => (
            <TabLink
              key={route.key}
              index={route.tabIndex}
              label={route.label}
              isActive={route.tabIndex === this.props.index}
              textColor={this.tabTextColorInterpolations[route.tabIndex]}
              onPress={this.handleTabPress}
              onLayout={this.handleTabLinkLayout}
            />
          )
        )}

      </View>
    )
  }
}

class TabLink extends React.PureComponent {

  @bind
  handlePress() {
    this.props.onPress(this.props.index)
  }

  @bind
  handleLayout(event) {
    this.props.onLayout(event, this.props.index)
  }

  render() {
    return (
      <TouchableOpacity
        onPress={this.handlePress}
        onLayout={this.handleLayout}
      >
        <View
          style={styles.tabContainer}
        >
          <Animated.Text
            style={{ color: this.props.textColor }}
          >
            {this.props.label}
          </Animated.Text>
        </View>
      </TouchableOpacity>
    )
  }
}


const styles = StyleSheet.create({
  scene: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  header: {
    marginTop: 20,
    height: 60,
    flexDirection: 'row',
    alignItems: 'center',
  },
  tabContainer: {
    marginHorizontal: 8,
    backgroundColor: 'transparent',
  },
  tabIndicator: {
    position: 'absolute',
    left: 0,
    top: 0,
    height: '100%',
    backgroundColor: 'tomato',
  },
})
```

Alright awesome. Now let's get into styling and add in just 1 more interpolation. And that will be `backgroundColor`.

First off, let's update our routes in `MyTabView` and also update our `renderScene` function.

```jsx

export default class MyTabView extends React.Component {
  //...
  state = {
    index: 0,
    routes: [
      { tabIndex: 0, key: '0', label: `Tab #1`, backgroundColor: '#A7B4CA' },
      { tabIndex: 1, key: '1', label: `Wide Tab #2`, backgroundColor: '#DBA09B' },
      { tabIndex: 2, key: '2', label: `#3`, backgroundColor: '#E7DF93' },
      { tabIndex: 3, key: '3', label: `Really Wide Tab #4`, backgroundColor: '#9FDCB0' },
    ],
  }
  //...
  @bind
  renderScene(props) {
    return (
      <View
        style={[
          styles.scene,
          {
            backgroundColor: props.route.backgroundColor,
          },
        ]}
      >
        <Text>{`Hi I'm ${props.route.label}`}</Text>
      </View>
    )
  }
  //...
}
```

I've chosen four colors for each scene and added them to the scene's View style.

Now let's update our tab indictor interpolation to also change color.

```jsx
class TabHeader extends React.Component {
  //...
  @bind
  setTabIndicatorInterpolations() {
    const tabIndices = Object.keys(this.tabMeasurements).map((key) => {
      return Number(key)
    })

    const tabPositions = tabIndices.map((tabIndex) => {
      return this.tabMeasurements[tabIndex].x
    })

    const tabWidths = tabIndices.map((tabIndex) => {
      return this.tabMeasurements[tabIndex].width
    })

    // New Array that maps over routes and puts the backgroundColor's within an array
    const tabColors = this.props.routes.map((route) => {
      return route.backgroundColor
    })

    const tabIndicatorTranslateX = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabPositions,
    })

    const tabIndicatorWidth = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabWidths,
    })

    // Interpolate those values just like we did with translateX and width
    const tabIndicatorBackgroundColor = this.positionAnimatedValue.interpolate({
      inputRange: tabIndices,
      outputRange: tabColors,
    })

    const tabIndicatorAnimatedStyle = {
      width: tabIndicatorWidth,
      transform: [{ translateX: tabIndicatorTranslateX }],
      // Add this interpolation to our animated style
      backgroundColor: tabIndicatorBackgroundColor,
    }

    this.setState({ tabIndicatorAnimatedStyle })
  }
  //...
}
```
Awesome. That's all we need to get the background color to animate on `position` change.

Now add just a little more style:

```jsx
const styles = StyleSheet.create({
  //...
  header: {
    marginTop: 20,
    paddingHorizontal: 8,
    backgroundColor: '#fff',
    height: 60,
    width: '100%',
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-around',
  },
  //...
  tabIndicator: {
    position: 'absolute',
    left: 0,
    top: 8,
    bottom: 8,
    borderRadius: 8,
  },
})
```

And bam, you've got a nice looking `TabViewAnimated`

