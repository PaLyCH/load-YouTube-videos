# Lazy load Adaptive YouTube videos with JavaScript

Lazy loading YouTube videos with vanilla JavaScript is a great way to improve the user experience on your website. It
allows you to load videos only when they need to be displayed in order to reduce page loading times.

Lazy loading YouTube videos with JavaScript can be done using Google API or without. The easiest way to do it without
using the API is to create a custom script that parses the video URL to identify the video ID and embeds the video into
an HTML iframe.

Let’s start!

## The starting HTML

For this use case, let’s start with a link that points to the actual video we want to embed. Before our JS loads, or if
it fails for some reason, visitors will still have a way to watch the adaptive video.

We’ll also include a [data-youtube] attribute that we can target with our JavaScript after it loads.

```html
<div class="embed-container">
    <a data-youtube href="https://www.youtube.com/watch?v=r7Pij8O1kzM" title="Kite safari in Egypt" data-loading="false">
        Watch "Kite safari in Egypt" by Kitepiter
    </a>
</div>
```

## Loading a thumbnail for video

When our JavaScript loads, we can progressively enhance our `[data-youtube]` links.

We’ll extract the video **ID** from the **URL**, add a thumbnail, and progressively enhance the link into a button for
semantic reasons.

First, let’s get all of our `[data-youtube]` links with the `document.querySelectorAll()` method.

```js
// Get all of the videos
let videos = document.querySelectorAll('[data-youtube]');

// Progressively enhance them
for (let video of videos) {

    // Get the video ID
    let id = new URL(video.href).searchParams.get('v');

    // Add the ID to the data-youtube attribute
    video.setAttribute('data-youtube', id);

    // Add a role of button
    video.setAttribute('role', 'button');

}
```

Ok, we’ll get the video thumbnail and add that to the link.

YouTube uses a consistent URL pattern, with the video ID in the path, for its video thumbnails. We’ll use the
Element.innerHTML property to add an image to the URL, and keep the descriptive text with it.

Here's the Official Google Documentation for [Youtube API](https://developers.google.com/youtube/v3/docs/thumbnails)

For the high quality version of the thumbnail use a url similar to this:

```html
https://i.ytimg.com/vi/<VIDEO_ID>/hqdefault.jpg
https://i.ytimg.com/vi_webp/<VIDEO_ID>/hqdefault.webp`
```

There is also a medium quality version of the thumbnail, using a url similar to the HQ:

```html
https://i.ytimg.com/vi/<VIDEO_ID>/mqdefault.jpg
https://i.ytimg.com/vi_webp/<VIDEO_ID>/mqdefault.webp
```

For the standard definition version of the thumbnail, use a url similar to this:

```html
https://i.ytimg.com/vi/<VIDEO_ID>/sddefault.jpg
https://i.ytimg.com/vi_webp/<VIDEO_ID>/sddefault.webp
```

For the maximum resolution version of the thumbnail use a url similar to this:

```html
https://i.ytimg.com/vi/<VIDEO_ID>/maxresdefault.jpg
https://i.ytimg.com/vi_webp/<VIDEO_ID>/maxresdefault.webp
```

Webp is a new standard, but You can use old one.

## Then, we’ll get the video thumbnail and add that to the link.

```js
// Get all of the videos
let videos = document.querySelectorAll('[data-youtube]');

// Progressively enhance them
for (let video of videos) {

    // Get the video ID
    let id = new URL(video.href).searchParams.get('v');

    // Add the ID to the data-youtube attribute
    video.setAttribute('data-youtube', id);

    // Add a role of button
    video.setAttribute('role', 'button');

    // Get the video title
    let title = video.getAttribute('title');

    let loading = video.getAttribute('data-loading');

    // Add a thumbnail
    if (loading === 'true') {
        video.innerHTML = '<img alt="' + title + '" src="https://i.ytimg.com/vi_webp/' + id + '/sddefault.webp" width="622" height="467" loading="lazy">' + video.textContent;
    } else {
        video.innerHTML = '<img alt="' + title + '" src="https://i.ytimg.com/vi_webp/' + id + '/sddefault.webp" width="622" height="467">' + video.textContent;
    }
    video.parentElement.classList.add("img");
}
```

## Dynamically injecting the YouTube player

### 1. Setup a click event listener.

```js
// Detect clicks on the video thumbnails
document.addEventListener('click', clickHandler);
```

Inside the [clickHandler()]() function, we’ll first get the parent `[data-youtube]` element for the clicked item (
the `event.target`) using the `Element.closest()` method.

If no matching link was found, we’ll use the return operator to end the callback function early. Otherwise, we’ll
use `event.preventDefault()` to stop the link from redirecting users away from the site.

```js/**
* Handle click events on the video thumbnails
* @param  {Event} event The event object
*/
function clickHandler (event) {

  // Get the video link
  let link = event.target.closest('[data-youtube]');
  if (!link) return;

  // Prevent the URL from redirecting users
  event.preventDefault();
  
  // Get the video ID
  let id = link.getAttribute('data-youtube');
  
  // Get the video title
  let title = link.getAttribute('title');
  
  // Create the player
  let player = link.parentElement;
  player.innerHTML = '<iframe width="560" height="315" src="https://www.youtube.com/embed/'+id+'?autoplay=1" title="'+title+'" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>';
  player.classList.remove("img");

  // Inject the player into the UI
  link.replaceWith(player);
}
```

## Create CSS style

```scss
.embed-container {
  position       : relative;
  padding-bottom : 56.25%;
  height         : 0;
  max-width      : 100%;
  margin-bottom  : 1em;

  &.img {
	padding-bottom : 0;
	height         : auto;
  }

  iframe {
	position : absolute;
	top      : 0;
	left     : 0;
	width    : 100%;
	height   : 100%;
	border   : 0;
  }

  [data-youtube] {
	display  : block;
	position : relative;

	&::before {
	  content       : ' ';
	  z-index       : 2;
	  width         : 62px;
	  height        : 44px;
	  background    : #FFF;
	  border-radius : 8px;
	  position      : absolute;
	  left          : 50%;
	  top           : 50%;
	  transform     : translate(-50%, -50%);
	}

	&::after {
	  content      : ' ';
	  z-index      : 3;
	  width        : 0;
	  height       : 0;
	  border-style : solid;
	  border-width : 16.5px 0 16.5px 33px;
	  border-color : transparent transparent transparent #FFF;
	  position     : absolute;
	  left         : 50%;
	  top          : 50%;
	  transform    : translate(-50%, -50%);
	}

	&:hover {
	  &::after {
		opacity : .8;
	  }
	}
  }

  img {
	margin    : 0 auto;
	display   : block;
	max-width : 100%;
	height    : auto
  }
}
```