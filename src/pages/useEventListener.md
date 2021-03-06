---
templateKey: post
title: useEventListener
date: "2019-03-27"
gist: https://gist.github.com/gragland/ae701852ae6159c712d860a946cd7ca0
sandbox: https://codesandbox.io/s/z64on3ypm
links:
  - url: https://github.com/donavon/use-event-listener
    name: donavon/use-event-listener
    description: Original source for this hook available as a library
code: "import { useState, useRef, useEffect, useCallback } from 'react';\r\n\r\n\/\/ Usage\r\nfunction App(){\r\n  \/\/ State for storing mouse coordinates\r\n  const [coords, setCoords] = useState({ x: 0, y: 0 });\r\n  \r\n  \/\/ Event handler utilizing useCallback ...\r\n  \/\/ ... so that reference never changes.\r\n  const handler = useCallback(\r\n    ({ clientX, clientY }) => {\r\n      \/\/ Update coordinates\r\n      setCoords({ x: clientX, y: clientY });\r\n    },\r\n    [setCoords]\r\n  );\r\n  \r\n  \/\/ Add event listener using our hook\r\n  useEventListener('mousemove', handler);\r\n  \r\n  return (\r\n    <h1>\r\n      The mouse position is ({coords.x}, {coords.y})\r\n    <\/h1>\r\n  );\r\n}\r\n\r\n\/\/ Hook\r\nfunction useEventListener(eventName, handler, element = window){\r\n  \/\/ Create a ref that stores handler\r\n  const savedHandler = useRef();\r\n  \r\n  \/\/ Update ref.current value if handler changes.\r\n  \/\/ This allows our effect below to always get latest handler ...\r\n  \/\/ ... without us needing to pass it in effect deps array ...\r\n  \/\/ ... and potentially cause effect to re-run every render.\r\n  useEffect(() => {\r\n    savedHandler.current = handler;\r\n  }, [handler]);\r\n\r\n  useEffect(\r\n    () => {\r\n      \/\/ Make sure element supports addEventListener\r\n      \/\/ On \r\n      const isSupported = element && element.addEventListener;\r\n      if (!isSupported) return;\r\n      \r\n      \/\/ Create event listener that calls handler function stored in ref\r\n      const eventListener = event => savedHandler.current(event);\r\n      \r\n      \/\/ Add event listener\r\n      element.addEventListener(eventName, eventListener);\r\n      \r\n      \/\/ Remove event listener on cleanup\r\n      return () => {\r\n        element.removeEventListener(eventName, eventListener);\r\n      };\r\n    },\r\n    [eventName, element] \/\/ Re-run if eventName or element changes\r\n  );\r\n};"
---

If you find yourself adding a lot of event listeners using `useEffect` you might consider moving that logic to a custom hook. In the recipe below we create a `useEventListener` hook that handles checking if `addEventListener` is supported, adding the event listener, and removal on cleanup. See it in action in the [CodeSandbox demo](https://codesandbox.io/s/z64on3ypm).
