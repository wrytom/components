# Vue Music Player Widget

A dynamic, UI/UX-centered music player widget built using Vue.js. This widget includes all the essential features for a smooth and interactive music listening experience. With album cover highlights, easy navigation through next/prev songs, and a progress bar that allows users to interact with the song's duration, this player brings an engaging and modern touch to your app.

## Features

- **Next/Previous Song**: Easily navigate between songs with intuitive next/prev buttons.
- **Album Cover Highlight**: The album cover dynamically updates to provide a visually appealing experience.
- **Duration Handling**: Display the current song duration and total duration, offering users insight into song progress.
- **Progress Bar Interactivity**: Click the progress bar to jump to any point in the song.

## Usage

The component is simply used with a songs prop, which is an array of objects:

```javascript
<MusicPlayer :songs="songs" />
```


```javascript
const songs = [
  { title: "Shokolad", artist: "Ostava", duration: "3:31", thumbnail: ThumbnailImage },
  { title: "Ponedelnik", artist: "Ostava", duration: "3:32", thumbnail: ThumbnailImage },
  { title: "President", artist: "Ostava", duration: "3:30", thumbnail: ThumbnailImage },
];
```

## Demo
![Music Player Demo](demo.gif)
