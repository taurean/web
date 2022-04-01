---
title: Managing the Transition CSS Property
date: 2019-06-07
tags:
  - CSS
layout: layouts/post.njk
---
The transition CSS property is something that I don’t see talked about often and yet it’s one of the more difficult places to keep things consistent. If you’re working in a larger codebase, especially with enterprise web software like me, it’s easy for things to become inconsistent and seem impossible to apply constraints to. I chalk this up to a few different reasons:

- in a situation where you need to add an additional transition property when one is already set, you need to remember to re-apply the previous transition property otherwise it will be overwritten.
- the transition property inherently includes every other CSS property
- the transition property handles a few different values that you might want to modify for different contexts (delay, duration, timing, property being transitioned, etc)

I have seen times where people defined a specific timing property in a variable and reused that, but I think that doesn’t provide enough support. If you’re trying to establish a consistent set of rules for exactly how other developers should be writing their CSS and you want to really control just how flexible the transition property should be, we’ll have to take things further. Setting only one option doesn’t do enough either, we have to consider any edge cases we might have. For instance, you can have a fast transition on a `color` property but if you want to add a transition to `transform` you might want to make that timing longer so that the change is still perceptible to a user.

Instead, I have a different approach using a map and a mixin. The first thing __we want to do is make writing out the transition property easier for other developers__ that you work with. Ideally, it should be as simple as this:

Fig 1 ([View Gist](https://gist.github.com/taurean/f1287b61209a06671f422fcb8650115d))
```css
.example-case {
  @include transition(opacity, color, transform);
  color: blue;
  opacity: 0.5;
  transform: translateX(0);
  
  &:hover,
  &:focus {
    color: dodgerblue;
    opacity: 1;
    transform: translatex(5px);
  }
}
```

Now that we have our end goal in mind, let’s set that up:

Fig 2 ([View Gist](https://gist.github.com/taurean/746024b86fba86547a16cf0ad0eaa935))
```scss
@mixin transition($properties...) {
    $transition-props: ();

    @each $property in $properties {
      $transition-props: append($transition-props, #{$property + ' ' + '100ms' + ' ' + 'ease'}, comma);
    }

    transition: $transition-props;
}
```

This lets us iterate through the list of CSS properties we want to animate and apply a `100ms` duration and `ease` for the timing-function. This is great! Developers can rely on a consistent mixin that manages duration and timing for them, all they need to do is specify the property they want to add a transition to.

Unfortunately we still have the problem of wanting different duration and timing properties for different CSS properties. Let’s take another look:

Fig 3 ([View Gist](https://gist.github.com/taurean/2fbcf385abfe774c287cede566ffdf8e))
```scss
$transition-props-map: (
  color: (
    duration: 75ms,
    timing: ease-in
  ),
  opacity: (
    duration: 100ms,
    timing: ease-in,
  ),
  transform: (
    duration: 175ms,
    timing: ease-in,
  )
);

@mixin transition($properties...) {
    $transition-props: ();

    @each $property in $properties {
      $duration: 150ms; // default value
      $timing: ease; // default value
      
      @if map-has-key($transition-props-map, $property) {
        
        $duration: map-get(map-get($transition-props-map, $property), duration);
        $timing: map-get(map-get($transition-props-map, $property), timing);
      }
        
      $transition-props: append($transition-props, #{$property + ' ' + $duration + ' ' + $timing}, comma);
    }

    transition: $transition-props;
}
```

Now we have a way to dictate custom durations and timing functions on a per-property level. Whenever a CSS property is used that isn’t already mentioned in the map, the default values are used instead. There _is_ one more thing I think we can add here. After all, there are some CSS properties that we should really avoid ever animating because of it’s browser performance. That isn’t something that’s easy to catch usually and now that we have a mixin for managing transitions we have the opportunity to manage that too.

Fig 4 ([View Gist](https://gist.github.com/taurean/4b07de8bceb1afa43e0a5102d674543f))
```scss
$transition-props-map: (
  color: (
    duration: 75ms,
    timing: ease-in
  ),
  opacity: (
    duration: 100ms,
    timing: ease-in,
  ),
  transform: (
    duration: 175ms,
    timing: ease-in,
  ),
  margin: banned,
  margin-top: banned,
  margin-right: banned,
  margin-bottom: banned,
  margin-left: banned,
  top: banned,
  right: banned,
  bottom: banned,
  left: banned,
  border: banned,
  outline: banned,
  border-size: banned,
  display: banned,
  position: banned,
  visibility: banned,
);

@mixin transition($properties...) {
    $transition-props: ();

    @each $property in $properties {
      $duration: 150ms; // default value
      $timing: ease; // default value
      
      @if map-has-key($transition-props-map, $property) {
        @if map-get($transition-props-map, $property) == 'banned' {
          @error 'The property `#{$property}` has poor performance or is not supported when used by the transition property. Please use a different property.';
        }
        
        $duration: map-get(map-get($transition-props-map, $property), duration);
        $timing: map-get(map-get($transition-props-map, $property), timing);
      }
        
      $transition-props: append($transition-props, #{$property + ' ' + $duration + ' ' + $timing}, comma);
    }

    transition: $transition-props;
}
```

And that’s it. Now, we can dictate custom attributes on a per CSS property basis, provide a helpful error message when developers include CSS properties with poor performance or isn’t supported (or use `@warn` if we don't want to be too harsh), and establish a malleable future-proofed approach to make the transition property easier to work with moving foward. Now that everything has come together let's look at that original example in the fig.1 gist using what we have built and see what the output looks like:

Fig 5 ([View Gist](https://gist.github.com/taurean/d9987776c59fdf6da83e697c57decd3e))

```scss
// The Map & Mixin
$transition-props-map: (
  color: (
    duration: 75ms,
    timing: ease-in
  ),
  opacity: (
    duration: 100ms,
    timing: ease-in,
  ),
  transform: (
    duration: 175ms,
    timing: ease-in-out,
  ),
  margin: banned,
  margin-top: banned,
  margin-right: banned,
  margin-bottom: banned,
  margin-left: banned,
  top: banned,
  right: banned,
  bottom: banned,
  left: banned,
  border: banned,
  outline: banned,
  border-size: banned,
  display: banned,
  position: banned,
  visibility: banned,
);

@mixin transition($properties...) {
    $transition-props: ();

    @each $property in $properties {
      $duration: 150ms; // default value
      $timing: ease; // default value
      
      @if map-has-key($transition-props-map, $property) {
        @if map-get($transition-props-map, $property) == 'banned' {
          @error 'The property `#{$property}` has poor performance or is not supported when used by the transition property. Please use a different property.';
        }
        
        $duration: map-get(map-get($transition-props-map, $property), duration);
        $timing: map-get(map-get($transition-props-map, $property), timing);
      }
        
      $transition-props: append($transition-props, #{$property + ' ' + $duration + ' ' + $timing}, comma);
    }

    transition: $transition-props;
}

// Example SCSS

.example-case {
  @include transition(opacity, color, transform);
  color: blue;
  opacity: 0.5;
  transform: translateX(0);
  
  &:hover,
  &:focus {
    color: dodgerblue;
    opacity: 1;
    transform: translatex(5px);
  }
}
```
```css
// Output
.example-case {
  transition: opacity 100ms ease-in, color 75ms ease-in, transform 175ms ease-in-out;
  color: blue;
  opacity: 0.5;
  transform: translateX(0);
}

.example-case:hover,
.example-case:focus {
  color: dodgerblue;
  opacity: 1;
  transform: translateX(5px);
}
```