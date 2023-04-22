# SteelEye Assignment Solution
## Q1) Explain what the simple List component does?

The basic List component is a React component that displays a list of selectable items by taking an array of items and rendering each item as a list item (li) element. A single list item is represented by the WrappedSingleListItem component, which receives four props: index, isSelected, onClickHandler, and text. The WrappedSingleListItem component sets the style prop of the li element to change the background color of the selected item to green and renders the text prop as the content of the list item.

The WrappedListComponent component takes an array of objects representing the items in the list and renders the entire list by mapping over the items array and rendering a WrappedSingleListItem component for each item. It maintains a selectedIndex state variable that keeps track of the currently selected item. When an item is clicked, the onClickHandler function is called with the index of the clicked item as its argument. This function sets the selectedIndex variable to the index of the clicked item, causing the background color of the selected item to change to green. The prop-state library is used here to make the code easiy debuggable as the props are passed down from the parent to child component.

The simple List component demonstrates how to use props, prop-state, state variables, and sub-components to create a reusable and functional component for displaying selectable lists in a React application.

## Q2) What problems / warnings are there with code? 
1. Inappropriate use of the setSelectedIndex function is being made. The first element of the array returned by the useState hook is the state value, and the second element is the function to update that specific state value. SetSelectedIndex is utilized as the state value rather than a function to change the state value in the WrappedListComponent. The appropriate code for this is : 

``` const [selectedIndex, setSelectedIndex] = useState(null); ```

2. Type checking for WrappedListComponent was implemented incorrectly as the PropTypes.arrayOf is a validator that checks that an array is made up of a certain type of element, not PropTypes.array. In this case, it is being used to check that the items prop is an array of objects. The data that is recieved in this component as props
is supposed to be an object and it contains 'items' array which is an array of objects which has a text property of string type.
``` WrappedListComponent.propTypes = {
  items: PropTypes.arrayOf(PropTypes.shape({      
    text: PropTypes.string.isRequired,
  })),
};   
```
3. In the WrappedSingleListItem component, the onClick handler is being called immediately when the component is rendered instead of when the component is clicked. As a solution, the onclick handler must be passed with a function reference to render it only when the list is clicked. The correct code should be:

``` onClick={() => onClickHandler(index)} ```

4. Every time the item's prop is changed, the WrappedListComponent component sets the selected index state to null. As a result, every time the list is updated, the component can lose its selection status. It would be preferable to only reset the selectedIndex state when the list has undergone a complete change, such as when the length of the items prop alters. The appropriate code sholud be : 

``` WrappedListComponent.defaultProps = {
  items: [],
};
```
5. The isSelected prop in the WrappedSingleListItem component should be a boolean but is being passed the selectedIndex state value, which inturn is a number. The appropruate code should be:

```isSelected={selectedIndex === index}```
6. The value of the unique key is not specified as a parameter for each child value in the map function. The Appropriate code should be :

``` {items.map((item, index) => (
        <SingleListItem
          key={index}
          onClickHandler={() => handleClick(index)}
          text={item.text}
          index={index}
          isSelected={selectedIndex===index}
        />
      ))}
```
## Q3) Please fix, optimize, and/or modify the component as much as you think is necessary.
### With respect to the changes suggested by me in the above question, here are a few more optimizations to the code provided :
1. Using object destructuring to extract the props at the start of the component definition will save the code from having to send each prop as a separate argument to the component. The code becomes easier to read and less prone to errors as a result.

2. Using the useMemo hook for the handleClick method: In the current implementation, even if the items prop hasn't changed, the handleClick function is regenerated each time the component is shown. If the array of things is huge, this may not be efficient. The method can be memoized using the useMemo hook so that it is only recreated when the item's prop changes.

3. Using the onClickHandler prop's useCallback hook: Similar to this, even if the function reference hasn't changed, the onClickHandler prop is rebuilt for each item in the list. The function reference can be memorised using the useCallback hook so that it is only recreated when the selectedIndex state changes.

### Keeping in mind these optimizations, the final working code will be : 
```
import React, { useState, useEffect, memo, useMemo, useCallback } from 'react';
import PropTypes from 'prop-types';

// Single List Item
const SingleListItem = memo(({ index, isSelected, onClickHandler, text }) => {
  return (
    <li
      style={{ backgroundColor: isSelected ? 'green' : 'red' }}
      onClick={() => onClickHandler(index)}
    >
      {text}
    </li>
  );
});

SingleListItem.propTypes = {
  index: PropTypes.number,
  isSelected: PropTypes.bool,
  onClickHandler: PropTypes.func.isRequired,
  text: PropTypes.string.isRequired,
};

// List Component
const ListComponent = memo(({ items }) => {
  const [selectedIndex, setSelectedIndex] = useState(null);

  useEffect(() => {
    setSelectedIndex(null);
  }, [items]);

  const handleClick = useCallback((index) => {
    setSelectedIndex(index);
  }, []);

  const memoizedHandleClick = useMemo(() => handleClick, [selectedIndex]);

  return (
    <ul style={{ textAlign: 'left' }}>
      {items.map((item, index) => (
        <SingleListItem
          key={index}
          onClickHandler={memoizedHandleClick}
          text={item.text}
          index={index}
          isSelected={selectedIndex === index}
        />
      ))}
    </ul>
  );
});

ListComponent.propTypes = {
  items: PropTypes.arrayOf(
    PropTypes.shape({
      text: PropTypes.string.isRequired,
    })
  ),
};

ListComponent.defaultProps = {
  items: [],
};

export default ListComponent;

```
### The Css can obviously be changed making this components look more visually appealing
```
.li{
    width: 30%;
    height: 9vh;
    text-align: center;
    display: flex;
    justify-content: center;
    align-items: center;
    list-style: none;
    margin-top: 2,6vh;
    margin-left: 36rem;
    color: white;
    cursor: pointer;
}
```
