# 🐛 Bug Report: TailwindCSS Hover Effects Not Working

## Description

In this Laravel + Mazer-based ERP project, we encountered an issue where TailwindCSS hover effects like `hover:scale-105` and `hover:shadow-lg` were not working as expected on dashboard module cards, even though Tailwind was correctly installed and configured.

## Environment

- Laravel 12
- Vite with `@vite('resources/css/app.css')`
- TailwindCSS 3.x
- Mazer UI Template
- Blade Templating

## Symptoms

- Adding Tailwind classes such as `transition`, `transform`, `hover:scale-105`, and `hover:shadow-lg` to `.card` elements produced no visible effect.
- No browser console error.
- CSS class attributes were applied in the DOM but had no visual impact.

## Cause

The Tailwind-powered `.card-hover-zoom` CSS class was defined **inside** a `@media print` block:

```blade
<style>
@media print {
    .card-hover-zoom {
        transition: transform 0.3s ease, box-shadow 0.3s ease;
    }

    .card-hover-zoom:hover {
        transform: scale(1.05);
        box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
    }
}
</style>
````

Because this block only applies styles when the page is printed, the hover effects were ignored under normal screen usage.

## Solution ✅

Move the hover-related CSS outside the `@media print` block:

```css
<style>
.card-hover-zoom {
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card-hover-zoom:hover {
    transform: scale(1.05);
    box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
}

@media print {
    /* ...printing-specific styles... */
}
</style>
```

## Recommendation

* Always keep screen-related styles (like hover animations, transforms, etc.) **outside** of `@media print`.
* Use Tailwind's utility classes directly if possible for better clarity.
* Consider reviewing Mazer’s default `.card` class styles, which may override some utility styles from Tailwind.

## Status

✅ Bug fixed and hover effects are now working correctly.
