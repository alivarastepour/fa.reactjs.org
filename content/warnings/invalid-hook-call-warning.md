---
title: Invalid Hook Call Warning
layout: single
permalink: warnings/invalid-hook-call-warning.html
---

  شما احتمالا به این دلیل اینجا هستید که خطای زیر را دریافت کرده اید:

 > Hooks can only be called inside the body of a function component

این خطا به سه دلیل متداول به جود می آید:

1. ممکن است نسخه ی React و React Dom شما نامتناسب باشند.
2. ممکن است از **[قوانین هوک ها](/docs/hooks-rules.html)** تبعیت نکرده باشید.
3. ممکن است **بیش از یک نسخه از React** در یک برنامه داشته باشید.

بیایید هر یک از موارد بالا را به صورت جداگانه بررسی کنیم.

## نامتناسب بودن نسخه ی React و React DOM {#mismatching-versions-of-react-and-react-dom}

ممکن است شما نسخه ای از ‍‍`react-dom`(<16.8.0) یا `react−native`(<0.59)را نصب کرده باشید که از هوک ها پشتیانی نمی کند. شما می توانید در پوشه برنامه خود با اجرای دستور `npm ls react-dom` یا دستور `npm ls react-native` در ترمینال از نسخه مورد استفاده خود مطلع شوید. وجود بیش از یک نسخه از آن ها در پوشه برنامه هم می تواند مشکل ایجاد کند.
## عدم تبعیت از قوانین هوک ها {#breaking-the-rules-of-hooks}

 تنها زمانی مجاز به فراخوانی هوک ها هستید که **ری‌اکت در حال رندر کردن یک function component باشد**.
* ✅ هوک ها را اول بدنه ی function component ها فراخوانی کنید.
* ✅ هوک ها را اول بدنه ی [custom Hook](/docs/hooks-custom.html) ها فراخوانی کنید

**درباره این موضوع در [قوانین هوک ها](/docs/hooks-rules.html) بیشتر مطالعه کنید.**

```js{2-3,8-9}
// function component
function Counter() {
  // ✅ استفاده مناسب
  const [count, setCount] = useState(0);
  // ...
}

// custom hook
function useWindowWidth() {
  // ✅ استفاده مناسب
  const [width, setWidth] = useState(window.innerWidth);
  // ...
}
```

فراخوانی هوک ها در موارد پایین پشتیبانی نمی شود:
* 🔴 در class component ها هوک ها را فراخوانی نکنید.
* 🔴 در event handler ها هوک ها را فراخوانی نکنید.
* 🔴 هوک ها را در توابعی که به `useMemo`, `useReducer` یا `useEffect` ارسال شده اند فراخوانی نکنید.

اگر از قوانین بالا تبعیت نکنید ممکن است به این خطا برخورد کنید.

```js{3-4,11-12,20-21}
function Bad1() {
   //event handlers
  function handleClick() {
    // 🔴 استفاده نا مناسب : هوک را به خارج این قسمت منتقل کنید
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad2() {
   //useMemo
  const style = useMemo(() => {
    // 🔴 استفاده نامناسب : هوک را به خارج این قسمت منتقل کنید
    const theme = useContext(ThemeContext);
    return createStyle(theme);
  });
  // ...
}

class Bad3 extends React.Component {
   //class component
  render() {
    // 🔴 استفاده نامناسب
    useEffect(() => {})
    // ...
  }
}
```

می توانید برای یافتن برخی ازین اشکالات از [`eslint-plugin-react-hooks` plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) استفاده نمایید.
>نکته
> 
>custom hook ها *ممکن است* دیگر هوک ها را فراخوانی کنند(هدف آن ها همین است!). این فراخوانی مشکلی ایجاد نمی کند چون custom hook ها خود نیز باید وقتی فراخوانی شوند که یک function component در حال رندر شدن است.

## استفاده از چند نسخه ری‌اکت {#duplicate-react}

برای اینکه هوک ها به طور صحیح کار کنند import react از کد برنامه شما باید با import react از داخل بسته `react-dom` از یک `module` باشند.
اگر import های `react` مربوط به دو شئ متفاوت شوند این خطا را می بینید. این ممکن است به این دلیل رخ داده باشد که شما به صورت اتفاقی دو بسته از `react` را داشته باشید.
اگر از `Node` برای مدیریت بسته ها استفاده می کنید می توانید این تست را در پوشه برنامه خود اجرا کنید:

    npm ls react

اگر بیش از یک `react` می بینید باید اشکال کار را پیدا کنید و `dependency tree` خود را اصلاح کنید. برای مثال ممکن است کتابخانه ای که استفاده می کنید `react` را اشتباها `dependency` تعریف کرده باشد(به جای `peer dependency`). تا زمانی که آن کتابخانه اصلاح شود [Yarn resolutions](https://yarnpkg.com/lang/en/docs/selective-version-resolutions/) یک راهکار احتمالی است.
هم چنین می توانید سعی کنید مشکل خود را به وسیله ریست کردن `development server` یا اضافه کردن چند `log` حل کنید. 
```js
// این قطعه کد را به فایل زیر اضافه کنید
// node_modules/react-dom/index.js
window.React1 = require('react');

// این قطعه کد را به فایل کامپوننت خود اضافه کنید
require('react-dom');
window.React2 = require('react');
console.log(window.React1 === window.React2);
```

اگر خروجی کد بالا `false` بود شما احتمالا دو نسخه از `react` دارید و باید دلیل آن را بیابید.
[This issue](https://github.com/facebook/react/issues/13991) شامل تعدادی از دلایل متداول است که برنامه نویسان دیگر به آن برخورده اند.
این مشکل ممکن است با استفاده از `npm link` یا نظایر آن پدید آید. در این صورت `bundler` شما ممکن است دو نسخه از `react` را شناسایی کند. یکی در پوشه برنامه و دیگری در پوشه کتابخانه. با فرض اینکه پوشه های `myapp` و `mylib` در یک پوشه یکسان هستند یک راهکار احتمالی اجرای `npm link ../myapp/node_modules/react` از `mylib` است. این دستور کتابخانه را مجبور به استفاده از نسخه `react` برنامه می کند.
>نکته
>
به طور کلی ری‌اکت استفاده از چند کپی مستقل در یک صفحه را پشتیبانی می‌کند.(مثلا یک برنامه و یک ابزار third party هر دو از آن استفاده کنند.)مشکل زمانی به جود می آید که `require('react')` در کامپوننت و نسخه react-dom که با آن رندر شده است به دو شئ متفاوت اشاره کند. 
## دلایل دیگر {#other-causes}

اگر هیچکدام از راه حل های بالا موثر نبود لطفا در [this issue](https://github.com/facebook/react/issues/13991) نظر بگذارید و ما سعی می کنیم به شما کمک کنیم. سعی کنید یک مثال بازسازی شده و کوچک بسازید - ممکن است در حین آن به مشکل خود پی ببرید!