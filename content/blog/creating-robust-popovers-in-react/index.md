---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Creating Robust Popovers in React using Material UI"
subtitle: ""
summary: "A tutorial on creating robust popovers using Material UI."
authors: []
tags: [React, Frontend, Typescript, TSX, Material, MUI]
categories: [React]
date: 2022-02-21T18:48:03-05:00
lastmod: 2022-02-21T18:48:03-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
Popovers are a crucial part of any modern web application. Simply put, a popover is a small modal that pops over the screen when the user hovers or clicks somewhere. Usually this comes in the form of a menu, but it could also be a drawer popping out from the left or right side of the screen.

When creating a web application, developers usually reach for a component library like [Material UI](https://mui.com/) or [Antd](https://ant.design/docs/react/introduce). Recently when playing around with Material (hereafter referred to as MUI), I reached for the [Popover](https://mui.com/components/popover/) component. Here, I’ll discuss the implementation details of this component as to uncover one crucial limitation that necessitated my own custom implementation.

# MUI Popover Implementation

The way the MUI popover component works is by first receiving an anchor element as a prop. The anchor element is usually whatever `div` the click or hover event is coming from; MUI uses this to determine where to place the popover. However, the popover itself does not get attached anywhere in the hierarchy of the anchor element. Instead, it gets attached to a new `div` which is a sibling of the root element (usually where `ReactDOM.render` is first called).

Let’s call this new `div` `root-sibling`. This new `div` will take up the entire width and height of the viewport. The reason for this is to register a custom click handler that will cause the popover to close whenever the user clicks anywhere on the screen outside of the popover. Without knowing specifics, I’m assuming this click handler is registered on mount.

This behavior introduces a crucial limitation - out of the box, we cannot allow the screen to continue to process hover events when the popover is open. For example, say we have a menu which opens a different popover for each menu item on hover. If we use the MUI popover out of the box, then those other menu items will be masked by `root-sibling` whenever any popover is open, and nothing will happen when the user hovers those elements.

Furthermore, setting `pointer-events: none` on `root-sibling` will not work, because this will cause the popover to just always stay open no matter where the user clicks (I believe the escape key would still work, though). So what are we to do if we want the hover behavior to be preserved?

# Custom Implementation

In order to continue to process other hover events *and* close the popover whenever the user clicks outside of it, I used a combination of the `pointer-events: none` styling and a custom click handler.

First, we should style the `Popover` MUI component with `pointer-events: none`. However, we need to take care to still process the click events inside of the popover, otherwise the user will not be able to interact with menu elements. So, on the highest-level `div` inside of the `Popover` component, make sure to style with `pointer-events: auto`.

Next, we need to register a custom click handler which will cause the popover to be closed whenever the user clicks outside of it. We will register this on the highest-level `div` inside of the component. We will first need a ref for that `div`, so that we can check where the click is. Use React’s [`useRef`](https://reactjs.org/docs/hooks-reference.html#useref) hook to generate that ref, and then pass it to that `div`:

```jsx
// PopoverContent.tsx
export default function PopoverContent() {
  const ref = useRef<HTMLElement | null>(null);

  return (
    <div ref={ref}>
      Content goes here
    </div>
  );
}
```

Then, on mount, we will register the click handler. We can grab the bounding rectangle for the ref, and then use that to check where the click is. If the click is outside of that rectangle, then we should close the popover, most likely using some prop. We also need to make sure to remove the click handler on unmount. We can do all of this inside of a [`useEffect`](https://reactjs.org/docs/hooks-reference.html#useeffect) hook:

```jsx
// PopoverContent.tsx
const { handleClose }: { handleClose: () => void } = props;

useEffect(() => {
  if (!ref || !ref.current) {
    return;
  }

  const rect: DOMRect = ref.current.getBoundingClientRect();
  const eventListener = (event: MouseEvent) => {
    if (
      event.clientX < rect.left ||
      event.clientX > rect.right ||
      event.clientY < rect.top ||
      event.clientY > rect.bottom
    ) {
      handleClose();
    }
  };

  document.addEventListener("mousedown", eventListener);

  return () => document.removeEventListener("mousedown", eventListener);
}, [ref, ref.current]);
```

And there we have it! Now we will be able to successfully process hover events while preserving the desired click behavior.