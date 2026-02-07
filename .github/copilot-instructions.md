# Copilot Instructions for Playground

## Project Overview
This is a collection of standalone, kid-friendly HTML web apps designed for mobile-first interaction. Each app is a self-contained single-file HTML page with embedded CSS and JavaScript. The project prioritizes simplicity, touch interaction, and mobile responsiveness.

## Architecture & Structure
- **Root**: [index.html](../index.html) serves as the portal/launcher for all apps
- **Apps**: Each subdirectory contains a complete standalone app:
  - [animal-sounds/](../animal-sounds/) - Interactive animal sound board with audio playback
  - [drawing/](../drawing/) - Touch-enabled canvas drawing app with PWA support

## Key Development Patterns

### Self-Contained Single-File Apps
All apps follow the pattern of embedding everything in one HTML file:
- Inline `<style>` blocks for CSS
- Inline `<script>` blocks for JavaScript
- Data URIs or inline manifests for PWA support (see [drawing/index.html](../drawing/index.html#L12-L31))

### PWA Manifest Pattern
Generate manifest files dynamically using blob URLs to avoid external JSON files:
```javascript
const manifest = {
    "name": "App Name",
    "short_name": "Short",
    "display": "standalone",
    "orientation": "any",
    "start_url": "./",
    "scope": "./",
    "background_color": "#667eea",
    "theme_color": "#667eea",
    "icons": [{
        "src": "data:image/svg+xml,%3Csvg...",
        "sizes": "512x512",
        "type": "image/svg+xml",
        "purpose": "any maskable"
    }]
};
const manifestBlob = new Blob([JSON.stringify(manifest)], {type: 'application/json'});
const manifestURL = URL.createObjectURL(manifestBlob);
document.getElementById('manifest').setAttribute('href', manifestURL);
```
- Embed SVG icons as data URIs (URL-encoded)
- Include both mobile meta tags AND manifest link
- Use `purpose: "any maskable"` for icon compatibility

### Mobile-First Touch Interaction
**Critical**: Always implement both mouse and touch events for all interactive elements:
```javascript
// Example from drawing app
canvas.addEventListener('mousedown', startDrawing);
canvas.addEventListener('touchstart', startDrawing);
canvas.addEventListener('mousemove', draw);
canvas.addEventListener('touchmove', draw);
```

Mobile meta tags are required in all apps:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
```

### Back Button Pattern
All sub-pages (not the root launcher) include a floating back button in the top-left corner for navigation:

**HTML** (place immediately after `<body>` opening tag):
```html
<button id="back-btn" class="back-button" aria-label="Go back" title="Go back">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
        <path d="M19 12H5M12 19l-7-7 7-7"/>
    </svg>
</button>
```

**CSS** (add to `<style>` block):
```css
/* Back Button */
.back-button {
    position: fixed;
    top: 20px;
    left: 20px;
    z-index: 1000;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.9);
    border: none;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    transition: transform 0.2s, box-shadow 0.2s, background 0.2s;
    -webkit-tap-highlight-color: transparent;
}

.back-button:active {
    transform: scale(0.9);
}

.back-button:hover {
    background: rgba(255, 255, 255, 1);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.back-button svg {
    width: 24px;
    height: 24px;
    stroke: #333;
}
```

**JavaScript** (add to `<script>` block):
```javascript
// Back button functionality
document.getElementById('back-btn').addEventListener('click', () => {
    window.history.back();
});
```

The back button is shared across all sub-app pages with consistent styling. It uses `window.history.back()` for standard browser navigation, working both on mobile and desktop.

### Responsive Design Philosophy
- Use `clamp()` for fluid typography: `font-size: clamp(2rem, 5vw, 3rem)`
- Grid layouts with `auto-fit` for flexibility: `grid-template-columns: repeat(auto-fit, minmax(140px, 1fr))`
- Mobile-specific breakpoints around 600-768px for layout adjustments
- Disable text selection and tap highlights: `-webkit-tap-highlight-color: transparent; user-select: none;`

### Animation & Interaction Feedback
Provide immediate visual feedback for user actions:
- Transform scale on active state: `.animal-card:active { transform: scale(0.95); }`
- CSS animations for engagement: `@keyframes bounce { ... }`
- Toggle classes for animation triggers (see [animal-sounds/index.html](../animal-sounds/index.html#L169-L173))

### Audio Pattern (Animal Sounds App)
Audio files are preloaded and managed in a dictionary:
```javascript
const audioElements = {};
Object.keys(soundUrls).forEach(animal => {
    audioElements[animal] = new Audio(soundUrls[animal]);
    audioElements[animal].preload = 'auto';
});
```
Reset `currentTime = 0` before playing to allow rapid repeated clicks.

### Canvas Pattern (Drawing App)
- Canvas resize must preserve existing drawing using temporary canvas buffer (see [drawing/index.html](../drawing/index.html#L335-L352))
- Use `touch-action: none` on canvas to prevent scrolling during drawing
- Implement eraser as white strokeStyle, not clearRect for better performance
- Hold-to-confirm pattern for destructive actions (3-second hold with progress bar)

### Styling Conventions
- Use CSS variables sparingly; prefer inline gradient definitions
- Gradient backgrounds for visual appeal: `linear-gradient(135deg, #667eea 0%, #764ba2 100%)`
- Border transitions indicate active state: `border-color: #ffeb3b`
- System fonts for better performance: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`

## Adding New Apps
1. Create new directory under root: `/new-app/`
2. Create single `index.html` with all embedded content
3. Add link to root [index.html](../index.html) launcher
4. Include mobile meta tags and touch event handlers
5. Include the back button (see [Back Button Pattern](#back-button-pattern)) to allow navigation back to the launcher
6. Test on actual mobile devices, not just responsive mode

## Asset Management
- **Audio files**: Store in app-specific `sounds/` subdirectory (see [animal-sounds/sounds/](../animal-sounds/sounds/))
- **Images**: Prefer emoji or SVG data URIs over separate image files
- **PWA icons**: Use inline SVG data URIs in manifest (see drawing app manifest generation)

## No Build Process
This project intentionally has no build tools, bundlers, or transpilation. Write vanilla JavaScript compatible with modern mobile browsers. No npm, no dependencies, no compilation step required.
