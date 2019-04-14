# react-spatial-navigation
[![npm version](https://badge.fury.io/js/%40noriginmedia%2Freact-spatial-navigation.svg)](https://badge.fury.io/js/%40noriginmedia%2Freact-spatial-navigation)

## Motivation
The main motivation to create this package was to bring the best Developer Experience and Performance when working with Key Navigation and React. Ideally you wouldn't want to have any logic to define the navigation in your app. It should be as easy as just to tell which components should be navigable. With this package all you have to do is to initialize it, wrap your components with the HOC and set initial focus. The spatial navigation system will automatically figure out which components to focus next when you navigate with the directional keys.

## Article
Will be published soon.

# Changelog
[CHANGELOG.md](https://github.com/NoriginMedia/react-spatial-navigation/blob/master/CHANGELOG.md)

# Table of Contents
* [Example](#example)
* [Installation](#installation)
* [Usage](#usage)
* [API](#api)
* [Development](#development)
* [TODOs](#todos)

# Example
![Spatial Navigation example](resources/images/spatial-nav-example.gif)

[Testbed Example](https://github.com/NoriginMedia/react-spatial-navigation/blob/master/src/App.js)

# Installation
```bash
npm i @noriginmedia/react-spatial-navigation --save
```

# Usage

## Initialization
```jsx
// Somewhere at the root of the app

import {initNavigation, setKeyMap} from '@noriginmedia/react-spatial-navigation';

initNavigation();

// Optional
setKeyMap({
  'left': 9001,
  'up': 9002,
  'right': 9003,
  'down': 9004,
  'enter': 9005
});
```

## Making component focusable
```jsx
import {withFocusable} from '@noriginmedia/react-spatial-navigation';

...

const FocusableComponent = withFocusable()(Component);
```

## Using config options
```jsx
import {withFocusable} from '@noriginmedia/react-spatial-navigation';

...

const FocusableComponent = withFocusable({
  propagateFocus: true,
  trackChildren: true,
  forgetLastFocusedChild: true
})(Component);
```

## Using props on focusable components
```jsx
import {withFocusable} from '@noriginmedia/react-spatial-navigation';

...

const FocusableComponent = withFocusable()(Component);

const ParentComponent = (props) => (<View>
  ...
  <FocusableComponent 
    propagateFocus
    trackChildren
    forgetLastFocusedChild
    focusKey={'FOCUSABLE_COMPONENT'}
    onEnterPress={props.onItemPress}
    onBecameFocused={props.onItemFocused}
  />
  ...
</View>);
```

## Using props inside wrapped components
### Basic usage
```jsx
import {withFocusable} from '@noriginmedia/react-spatial-navigation';

const Component = ({focused, setFocus}) => (<View>
  <View style={focused ? styles.focusedStyle : styles.defaultStyle} />
  <TouchableOpacity 
    onPress={() => {
      setFocus('SOME_ANOTHER_COMPONENT');
    }}
  />
</View>);

const FocusableComponent = withFocusable()(Component);
```

### Setting initial focus on child component, tracking children
```jsx
import React, {PureComponent} from 'react';
import {withFocusable} from '@noriginmedia/react-spatial-navigation';

...

class Menu extends PureComponent {
  componentDidMount() {
    // this.props.setFocus(); // If you need to focus first child automatically
    this.props.setFocus('MENU-6'); // If you need to focus specific item that you know focus key of
  }

  render() {
    return (<View style={hasFocusedChild ? styles.menuExpanded : styles.defaultStyle}>
      <MenuItemFocusable />
      <MenuItemFocusable />
      <MenuItemFocusable />
      <MenuItemFocusable />
      <MenuItemFocusable />
      <MenuItemFocusable focusKey={'MENU-6'} />
    </View>);
  }
}

const MenuFocusable = withFocusable({
  trackChildren: true,
  propagateFocus: true
})(Menu);
```

# API

## Top level
### `initNavigation`: function
Function that needs to be called to enable Spatial Navigation system and bind key event listeners.
Accepts [initConfig](#initConfig) as a param.

```jsx
initNavigation({
  debug: true,
  visualDebug: true
})
```

## Initialization Config
### `debug`: boolean
Enable console debugging

* **false (default)**
* **true**

### `visualDebug`: boolean
Enable visual debugging (all layouts, reference points and siblings refernce points are printed on canvases)

* **false (default)**
* **true**

### `setKeyMap`: function
Function to set custom key codes.
```jsx
setKeyMap({
  'left': 9001,
  'up': 9002,
  'right': 9003,
  'down': 9004,
  'enter': 9005
});
```

### `withFocusable`: function
Main HOC wrapper function. Accepts [config](#config) as a param.
```jsx
const FocusableComponent = withFocusable({...})(Component);
```

## Config
### `propagateFocus`: boolean
Determine whether to automatically propagate focus to child focusable component when this component gets focused.

* **false (default)**
* **true**

### `trackChildren`: boolean
Determine whether to track when any child component is focused. Wrapped component can rely on `hasFocusedChild` prop when this mode is enabled. Otherwise `hasFocusedChild` will be always `false`.

* **false (default)** - Disabled by default because it causes unnecessary render call when `hasFocusedChild` changes
* **true**

### `forgetLastFocusedChild`: boolean
Determine whether this component should not remember the last focused child components. By default when focus goes away from the component and then it gets focused again, it will focus the last focused child. This functionality is enabled by default.

* **false (default)**
* **true**

## Props that can be applied to HOC
All these props are optional.

### `propagateFocus`: boolean
Same as in [config](#config).

### `trackChildren`: boolean
Same as in [config](#config).

### `forgetLastFocusedChild`: boolean
Same as in [config](#config).

### `focusKey`: string
String that is used as a component focus key. Should be **unique**, otherwise it will override previously stored component with the same focus key in the Spatial Navigation service storage of focusable components. If this is not specified, the focus key will be generated automatically.

### `onEnterPress`: function
Callback function that is called when the item is currently focused and Enter (OK) key is pressed.

Payload:
All the props passed to HOC is passed back to this callback. Useful to avoid creating callback functions during render.

```jsx
const onPress = ({prop1, prop2}) => {...};

...
<FocusableItem 
  prop1={111}
  prop2={222}
  onPress={onPress}
/>
...
```

### `onBecameFocused`: function
Callback function that is called when the item becomes focused directly or during propagation of the focus to the children components. For example when you have nested tree of 5 focusable components, each of which has `propagateFocus`, this callback will be called on every level of propagation.

Payload:
Component layout object is passed as a first param. All the component props passed back to this callback. Useful to avoid creating callback functions during render. `x` and `y` are relative coordinates to parent DOM (**not the Focusable parent**) element. `left` and `top` are absolute coordinates on the screen.

```jsx
const onFocused = ({width, height, x, y, top, left}, {prop1, prop2}) => {...};

...
<FocusableItem 
  prop1={111}
  prop2={222}
  onBecameFocused={onFocused}
/>
...
```

## Props passed to Wrapped Component
### `focusKey`: string
Focus key that represents the focus key that was applied to HOC component. Might be `null` when not set. It is recommended to not rely on this prop ¯\\\_(ツ)_/¯

### `realFocusKey`: string
Focus key that is either the `focusKey` prop of the HOC, or automatically generated focus key like `sn:focusable-item-23`.

### `parentFocusKey`: string
Focus key of the parent component. If it is a top level focusable component, this prop will be `SN:ROOT`

### `focused`: boolean
Whether component is currently focused. It is only `true` if this exact component is focused, e.g. when this component propagates focus to child component, this value will be `false`.

### `hasFocusedChild`: boolean
This prop indicates that the component currently has some focused child on any depth of the focusable tree.

### `setFocus`: function
This method sets the focus to another component (when focus key is passed as param) or steals the focus to itself (when used w/o params). It is also possible to set focus to a non-existent component, and it will be automatically picked up when component with that focus key will get mounted. This preemptive setting of the focus might be useful when rendering lists of data. You can assign focus key with the item index and set it to e.g. first item, then as soon as it will be rendered, that item will get focused.

```jsx
setFocus(); // set focus to self
setFocus('SOME_COMPONENT'); // set focus to another component if you know its focus key
```

### `pauseSpatialNavigation`: function
This function pauses key listeners. Useful when you need to temporary disable navigation. (e.g. when player controls are hidden during video playback and you want to bind the keys to show controls again).

### `resumeSpatialNavigation`: function
This function resumes key listeners if it was paused with [pauseSpatialNavigation](#pauseSpatialNavigation-function)

# Development
## Dev environment
This library is using Parcel to serve the web build.

To run the testbed app locally:
```
npm start
```
This will start local server on `localhost:1234`

Source code is in `src/App.js`

## Dev notes
### General notes
* Focusable component are stored as a Tree structure. Each component has the reference to its parent as `parentFocusKey`.
* Current algorithm calculates distance between the border of the current component in the direction of key press to the border of the next component.

### `withFocusable` HOC
* `realFocusKey` is created once at component mount in `withStateHandlers`. It either takes the `focusKey` prop value or is automatically generated.
* `setFocus` method is bound with the current component `realFocusKey` so you can call it w/o params to focus component itself. Also the behaviour of this method can be described as an *attempt to set the focus*, because even if the target component doesn't exist yet, the target focus key will be stored and the focus will be picked up by the component with that focus key when it will get mounted.
* `parentFocusKey` is propagated to children components through context. This is done because the focusable components tree is not necessary the same as the DOM tree.
* On mount component adds itself to `spatialNavigation` service storage of all focusable components.
* On unmount component removes itself from the service. 

### `spatialNavigation` Service
* New components are added in `addFocusable`and removed in `removeFocusable`
* Main function to change focus is `setFocus`. First it decides next focus key (`getNextFocusKey`), then set focus to the new component (`setCurrentFocusedKey`), then the service updates all components that has focused child and finally updates layout (coordinates and dimensions) for all focusable component.
* `getNextFocusKey` is used to determine the good candidate to focus when you call `setFocus`. This method will either return the target focus key for the component you are trying to focus, or go down by the focusable tree and select the best child component to focus. This function is recoursive and going down by the focusable tree.
* `smartNavigate` is similar to the previous one, but is called in response to a key press event. It tries to focus the best sibling candidate in the direction of key press, or delegates this task to a focusable parent, that will do the same attempt for its sibling and so on.

## Contributing
Please follow the [Contribution Guide](https://github.com/NoriginMedia/react-spatial-navigation/blob/master/CONTRIBUTING.md)

# TODOs
- [ ] Get rid of `propagateFocus`, because it is used in 99% of the times when component has children
- [ ] Unit tests
- [ ] Implement more advanced coordination calculation [algorithm](https://developer.mozilla.org/en-US/docs/Mozilla/Firefox_OS_for_TV/TV_remote_control_navigation#Algorithm_design).
- [ ] Refactor with React Hooks instead of recompose.
- [ ] Implement HOC for react-native tvOS and AndroidTV components.
- [ ] Add custom navigation logic per component. I.e. possibility to override default decision making algorithm and decide where to navigate next based on direction.
- [ ] Add "preferable focused component" feature for components with children. By default it's first element, but it is useful to customize this behaviour.
- [ ] Implement mouse support. On some TV devices (or in the Browser) it is possible to use mouse-like input, e.g. magic remote in LG TVs. This system should support switching between "key" and "pointer" modes and apply "focused" state accordingly.

---

# License
**MIT Licensed**
