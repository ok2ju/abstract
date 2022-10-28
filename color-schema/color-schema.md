# Color Schema

Manage color schema of your application.

# Light/Dark schema auto detection

- [prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme) - CSS media feature is used to detect if a user has requested light or dark color themes. A user indicates their preference through an operating system setting (e.g. light or dark mode) or a user agent setting.

- [matchMedia()](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia) - method returns a new `MediaQueryList` object that can then be used to determine if the document matches the media query string, as well as to monitor the document to detect when it matches (or stops matching) that media query.

```js
useEffect(() => {
  const isDarkSchema = window.matchMedia("(prefers-color-scheme: dark)");

  const switchTheme = (event: MediaQueryListEvent) => {
    if (event.matches) {
      setTheme(THEMES.dark);
    } else {
      setTheme(THEMES.light);
    }
  };

  isDarkSchema.addEventListener("change", switchTheme);

  return () => {
    isDarkSchema.removeEventListener("change", switchTheme);
  };
}, []);
```

## Sources

- [matchMedia](https://developer.mozilla.org/en-US/docs/Web/API/Window/matchMedia)
- [prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)
