# Useful Code Snippets

## Index

<ol>
<li>NextJS / Styled Components / TypeScript</li>
<li>Kill Port In Use (Unix)</li>
</ol>

### NextJS / Styled Components / TypeScript

1. Install `babel-plugin-styled-components`

```shell
npm i -D babel-plugin-styled-components
```

2. Create a `.babelrc` file at the root of the directory and add:

```json
{
  "presets": ["next/babel"],
  "plugins": [["styled-components", { "ssr": true }]]
}
```

3. Create a `_document.tsx` file in the `/pages` directory and add:

```javascript
import Document, {
  DocumentContext,
  Head,
  Html,
  Main,
  NextScript,
} from "next/document";
import { ServerStyleSheet } from "styled-components";

export default class AppDocument extends Document {
  static async getInitialProps(ctx: DocumentContext) {
    const sheet = new ServerStyleSheet();
    const originalRenderPage = ctx.renderPage;

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: (App) => (props) =>
            sheet.collectStyles(<App {...props} />),
        });

      const initialProps = await Document.getInitialProps(ctx);
      return {
        ...initialProps,
        styles: [
          <>
            {initialProps.styles}
            {sheet.getStyleElement()}
          </>,
        ],
      };
    } finally {
      sheet.seal();
    }
  }

  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}

```

### Kill Port In Use

```shell
lsof -i tcp:<PORT NUMBER>
kill -9 <PID>
```

### Typescript hasOwnProperty Typed
```javascript
function hasOwnProperty<X extends {}, Y extends PropertyKey>(
  obj: X,
  prop: Y
): obj is X & Record<Y, unknown> {
  return obj.hasOwnProperty(prop);
}
```

### useMediaQuery Hook

#### Hook
```javascript
import * as React from "react";

export interface IBreakpoints {
  xs: number;
  sm: number;
  md: number;
  lg: number;
  xl: number;
  xxl: number;
}

const initialBreakpoints: IBreakpoints = {
  xs: 0,
  sm: 576,
  md: 768,
  lg: 992,
  xl: 1200,
  xxl: 1400,
};

export const useMediaQuery = (
  values: any[],
  breakpoints: Partial<IBreakpoints> = initialBreakpoints
) => {
  const [initialLoad, setInitialLoad] = React.useState(true);
  const [value, setValue] = React.useState(null);
  breakpoints = { ...initialBreakpoints, ...breakpoints };

  /** handle matching values */
  const handleValueMatch = React.useCallback(() => {
    const breakpointEntries = Object.entries(breakpoints);

    // ensure available values matches number of breakpoints
    let valuesArr = [];
    if (values.length < breakpointEntries.length) {
      valuesArr = [...values];
      const diff = breakpointEntries.length - values.length;
      Array.from(Array(diff)).forEach((_) =>
        valuesArr.push(values[values.length - 1])
      );
    } else if (values.length > breakpointEntries.length) {
      valuesArr = [...values];
      valuesArr.length = breakpointEntries.length;
    } else {
      valuesArr = [...values];
    }

    // get window width on resize
    let width = window.innerWidth;

    // get value at current breakpoint
    const matches = breakpointEntries.filter(([, value]) => width >= value);
    const match = matches[matches.length - 1][0];
    const matchIndex = breakpointEntries
      .map((entry) => entry[0])
      .indexOf(match);
    setValue(valuesArr[matchIndex]);
  }, [breakpoints, values]);

  /** handle first load */
  React.useEffect(() => {
    if (!initialLoad) return;
    handleValueMatch();
    setInitialLoad(false);
  }, [initialLoad, handleValueMatch]);

  /** handle resize */
  React.useEffect(() => {
    if (!window) return;
    const listener = () => {
      handleValueMatch();
    };

    window.addEventListener("resize", listener);
    return () => window.removeEventListener("resize", listener);
  }, [handleValueMatch]);

  return value;
};
```

#### Usage
```javascript
// using default breakpoints
const values = useMediaQuery([0, 1, 2, 3, 4, 5]);

// customising breakpoints
const values = useMediaQuery([0, 1, 2, 3, 4, 5], { xs: 0, sm: 320, md: 550, lg: 880, xl: 1200, xxl: 1440 });

// useMediaQuery extends the last value in the array across all remaining breakpoints
const values = useMediaQuery([0, 1, 2]); // equivalent to [0, 1, 2, 3, 4, 5]

// customising specific breakpoints only
const values = useMediaQuery([0, 1, 2], { md: 550, xxl: 1440 });
```
