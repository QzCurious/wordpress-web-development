# WordPress Custom Templates with Alpine.js and TailwindCSS

## Overview

This guide demonstrates how to create WordPress plugins that use custom page templates with Alpine.js and TailwindCSS integration. This approach allows you to:

1. Create interactive pages using Alpine.js without modifying your active theme
2. Implement utility-first styling with TailwindCSS that adapts to WordPress Global Styles
3. Allow WordPress users to edit content using familiar Gutenberg blocks
4. Implement server-side rendering for SEO benefits with client-side interactivity
5. Create a solution compatible with any WordPress theme
6. Leverage modern build tools for asset management

## Complete Workflow Overview

This section provides a comprehensive overview of how all the components of an Alpine.js and TailwindCSS WordPress plugin work together, from registration to rendering.

### Implementation Flowchart

```
┌─────────────────────────────┐
│ WordPress Plugin Activation │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Register Custom Template    │◄───────────┐
└──────────────┬──────────────┘            │
               ▼                           │
┌─────────────────────────────┐            │
│ Enqueue Alpine.js,          │            │ WordPress
│ TailwindCSS & Assets        │            │ Initialization
└──────────────┬──────────────┘            │ Process
               ▼                           │
┌─────────────────────────────┐            │
│ Create Default Page         │            │
│ (Optional)                  │            │
└──────────────┬──────────────┘            │
               │                           │
               └───────────────────────────┘


┌─────────────────────────────┐
│ User Visits Page            │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ WordPress Processes Request │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐     ┌─────────────────────────┐
│ template_include Filter     │────►│ Check Template          │
└──────────────┬──────────────┘     │ Conditions              │
               │                    └───────────┬─────────────┘
               ▼                                │
┌─────────────────────────────┐                 │
│ Load Custom Template        │◄────────────────┘
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Template Processing         │
├─────────────────────────────┤
│ 1. get_header()             │
│ 2. Prepare Data             │
│ 3. Render HTML (SSR)        │
│ 4. Add Alpine.js Directives │
│ 5. Add TailwindCSS Classes  │
│ 6. get_footer()             │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Browser Receives HTML       │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Alpine.js Initializes       │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Interactive UI Ready        │
└─────────────────────────────┘
```

### How the Parts Work Together

1. **Plugin Registration**

   - Your plugin registers a custom page template using `add_filter('theme_page_templates')`
   - It adds a template loader using `add_filter('template_include')`
   - It enqueues Alpine.js, TailwindCSS styles, and other assets using `wp_enqueue_scripts`

2. **Template Selection**

   - When a user visits a page, WordPress checks which template to use
   - If your template is selected (either manually or via conditionals), WordPress runs your template loader
   - Your custom template file is loaded instead of the theme's default template

3. **Server-Side Rendering**

   - Your template prepares data using WordPress functions
   - It renders the HTML structure server-side using PHP
   - It includes Alpine.js directives for interactive elements
   - It applies TailwindCSS utility classes that respect WordPress Global Styles
   - All content is rendered by the server, with Alpine.js only handling interactivity

4. **Client-Side Interactivity**
   - When the page loads in the browser, Alpine.js initializes
   - It adds interactivity to elements with x-data and other directives
   - The user sees a fully rendered page that becomes interactive without any layout shifts

## Project Structure

Here's the recommended file structure for your plugin:

```
your-plugin/
├── build/                       # Built assets (generated via build process)
│   ├── index.js                 # Bundled JavaScript including Alpine.js
│   ├── index.asset.php          # Dependency and version data for WP
│   └── style.css                # Compiled CSS with TailwindCSS
├── src/                         # Source code
│   ├── js/
│   │   ├── index.js             # Main JS entry point
│   │   └── alpine-init.js       # Alpine.js initialization
│   └── scss/
│       └── style.scss           # Main SCSS file with TailwindCSS imports
├── templates/                   # WordPress page templates
│   ├── alpine-template.php      # Basic page template
│   ├── home-template.php        # Home page template
│   └── archive-template.php     # Archive template
├── includes/                    # PHP includes
│   └── template-functions.php   # Template helper functions
├── your-plugin.php              # Main plugin file
├── tailwind.config.js           # TailwindCSS configuration
├── postcss.config.js            # PostCSS configuration
├── package.json                 # NPM configuration
└── README.md                    # Documentation
```

## Development Environment Setup

### NPM Configuration

Create a `package.json` file in your plugin root:

```json
{
  "name": "your-plugin",
  "version": "1.0.0",
  "description": "WordPress plugin with Alpine.js and TailwindCSS custom templates",
  "main": "build/index.js",
  "scripts": {
    "build": "wp-scripts build src/js/index.js --output-path=build",
    "start": "wp-scripts start src/js/index.js --output-path=build",
    "lint:css": "wp-scripts lint-style",
    "lint:js": "wp-scripts lint-js"
  },
  "author": "Your Name",
  "license": "GPL-2.0-or-later",
  "devDependencies": {
    "@wordpress/scripts": "^25.0.0",
    "tailwindcss": "^3.3.0",
    "postcss": "^8.4.20",
    "autoprefixer": "^10.4.14"
  },
  "dependencies": {
    "alpinejs": "^3.12.0"
  }
}
```

The `@wordpress/scripts` package provides optimized configurations for bundling assets including:

- Webpack for bundling
- Babel for JavaScript transpilation
- PostCSS and SASS for CSS processing
- TailwindCSS integration via PostCSS

### Understanding WordPress Global Style Variables

Before configuring TailwindCSS to work with WordPress Global Styles, it's important to understand where these CSS variables come from and how to handle them reliably:

#### How WordPress CSS Variables Are Created

The CSS variables like `--wp--preset--color--primary` are part of WordPress's Global Styles system, which was introduced with Full Site Editing. These variables are injected into the site in different ways:

1. **Block Themes (FSE themes)**:
   - Variables are automatically generated from the theme's `theme.json` file
   - Defines colors, typography, spacing and other design elements
   - Automatically converted to CSS variables in the frontend

2. **Classic Themes**:
   - May not have these variables available by default
   - Can add limited theme.json support even in classic themes
   - Some plugins may add their own CSS variables

Here's an example of how WordPress generates these variables from theme.json:

```json
// theme.json example
{
  "version": 2,
  "settings": {
    "color": {
      "palette": [
        {
          "slug": "primary",
          "color": "#0073aa",
          "name": "Primary"
        },
        {
          "slug": "secondary",
          "color": "#23282d",
          "name": "Secondary"
        }
      ]
    }
  }
}
```

This automatically creates CSS variables:
```css
:root {
  --wp--preset--color--primary: #0073aa;
  --wp--preset--color--secondary: #23282d;
}
```

#### WordPress Developer Documentation References

According to the [official WordPress Block Editor Handbook](https://developer.wordpress.org/block-editor/how-to-guides/themes/global-settings-and-styles/), WordPress automatically generates CSS Custom Properties for preset values:

```css
/* CSS Custom Properties for the preset values */
body {
  --wp--preset--<PRESET_TYPE>--<PRESET_SLUG>: <DEFAULT_VALUE>;
  --wp--preset--color--pale-pink: #f78da7;
  --wp--preset--font-size--large: 36px;
  /* etc. */
}
```

The [WordPress Theme Handbook: Color section](https://developer.wordpress.org/themes/global-settings-and-styles/settings/color/) also states:

> "WordPress will automatically generate CSS custom properties for each of your presets in the form of `--wp--preset--{type}--{slug}`. So a color palette preset with the slug of `contrast` will become `--wp--preset--color--contrast`."

Importantly, the documentation notes that these CSS variables are generated even if you disable the default WordPress color palette:

> "It's important to note that, even if these settings are disabled, WordPress will still generate the CSS custom properties for its presets. This is for backward-compatibility and so that users do not lose colors, gradients, or duotone filters that they have previously chosen when using another theme."

This explains why these variables are always available for you to use in your TailwindCSS configuration, even when working with themes that don't explicitly define them.

#### Handling Missing CSS Variables

Since these variables may not exist in all WordPress environments, you should implement a strategy to handle them:

### TailwindCSS Configuration

Create a `tailwind.config.js` file to leverage WordPress Global Styles:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./templates/**/*.php", "./src/**/*.js"],
  theme: {
    extend: {
      colors: {
        // Map WordPress global style colors to Tailwind
        "wp-primary": "var(--wp--preset--color--primary, #0073aa)",
        "wp-secondary": "var(--wp--preset--color--secondary, #23282d)",
        "wp-tertiary": "var(--wp--preset--color--tertiary, #6c757d)",
        "wp-background": "var(--wp--preset--color--background, #fff)",
        "wp-foreground": "var(--wp--preset--color--foreground, #333)",
      },
      fontSize: {
        // Map WordPress global font sizes
        "wp-small": "var(--wp--preset--font-size--small, 0.875rem)",
        "wp-medium": "var(--wp--preset--font-size--medium, 1rem)",
        "wp-large": "var(--wp--preset--font-size--large, 1.125rem)",
        "wp-x-large": "var(--wp--preset--font-size--x-large, 1.5rem)",
      },
      spacing: {
        // WordPress spacing variables
        "wp-20": "var(--wp--style--block-gap, 1rem)",
        "wp-30": "var(--wp--custom--spacing--small, 1.25rem)",
        "wp-40": "var(--wp--custom--spacing--medium, 2rem)",
        "wp-50": "var(--wp--custom--spacing--large, 4rem)",
      },
    },
  },
  plugins: [],
};
```

Create a `postcss.config.js` file:

```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### Main SCSS File

Create `src/scss/style.scss`:

```scss
/* Tailwind directives */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom styles using WordPress global style variables */
@layer components {
  .wp-container {
    @apply px-4 mx-auto max-w-screen-xl;
    max-width: var(--wp--style--global--content-size, 1200px);
  }

  .wp-button {
    @apply px-4 py-2 rounded;
    background-color: var(--wp--preset--color--primary, #0073aa);
    color: var(--wp--preset--color--background, #fff);
  }
}
```

### Main JavaScript Entry Point

Create `src/js/index.js`:

```javascript
// Import Alpine.js
import Alpine from "alpinejs";

// Import styles with Tailwind
import "../scss/style.scss";

// Make Alpine globally available for debugging
window.Alpine = Alpine;

// Initialize Alpine
document.addEventListener("DOMContentLoaded", () => {
  Alpine.start();
});
```

### Install Dependencies

```bash
npm install
```

## Custom Template Registration

### Main Plugin File

Create `your-plugin.php` to register and load custom templates:

```php
<?php
/**
 * Plugin Name: Alpine.js and TailwindCSS Template
 * Description: A plugin that integrates Alpine.js and TailwindCSS with WordPress using custom page templates
 * Version: 1.0.0
 * Author: Your Name
 * Text Domain: alpine-tailwind-template
 */

// Exit if accessed directly
if (!defined('ABSPATH')) {
    exit;
}

// Register custom page templates
add_filter('theme_page_templates', 'add_alpine_tailwind_page_template');
function add_alpine_tailwind_page_template($templates) {
    $templates['alpine-template.php'] = 'Alpine.js and TailwindCSS Application';
    return $templates;
}

// Load template from plugin
add_filter('template_include', 'load_alpine_tailwind_page_template');
function load_alpine_tailwind_page_template($template) {
    if(get_page_template_slug() === 'alpine-template.php') {
        $template = plugin_dir_path(__FILE__) . 'templates/alpine-template.php';
    }
    return $template;
}

// Enqueue built assets
add_action('wp_enqueue_scripts', 'enqueue_alpine_tailwind_assets');
function enqueue_alpine_tailwind_assets() {
    if (is_page() && get_page_template_slug() === 'alpine-template.php') {
        $asset_file = include plugin_dir_path(__FILE__) . 'build/index.asset.php';

        // Enqueue the built JS with dependencies
        wp_enqueue_script(
            'alpine-tailwind-app',
            plugin_dir_url(__FILE__) . 'build/index.js',
            $asset_file['dependencies'],
            $asset_file['version'],
            true
        );

        // Enqueue the built CSS
        wp_enqueue_style(
            'alpine-tailwind-styles',
            plugin_dir_url(__FILE__) . 'build/style.css',
            array(),
            $asset_file['version']
        );
    }
}

// Create page on activation
register_activation_hook(__FILE__, 'create_alpine_tailwind_page');
function create_alpine_tailwind_page() {
    $page_check = get_page_by_path('alpine-tailwind-app');
    if (!$page_check) {
        $page = array(
            'post_title'    => 'Alpine.js and TailwindCSS Application',
            'post_name'     => 'alpine-tailwind-app',
            'post_status'   => 'publish',
            'post_type'     => 'page',
            'comment_status' => 'closed',
            'ping_status'   => 'closed',
        );

        $page_id = wp_insert_post($page);
        if ($page_id) {
            update_post_meta($page_id, '_wp_page_template', 'alpine-template.php');
        }
    }
}
```

## Creating the Template File

Create `templates/alpine-template.php`:

```php
<?php
/**
 * Template Name: Alpine.js and TailwindCSS Application
 * Description: A custom template for Alpine.js applications with TailwindCSS styling
 */

// Get header
get_header();

// Prepare data for Alpine.js
$page_data = array(
    'title' => get_the_title(),
    'id' => get_the_ID(),
    'author' => get_the_author_meta('display_name'),
    'date' => get_the_date()
);

// Site info
$site_info = array(
    'name' => get_bloginfo('name'),
    'description' => get_bloginfo('description'),
    'url' => home_url()
);
?>

<!-- Alpine.js Application with TailwindCSS -->
<div class="alpine-app bg-wp-background text-wp-foreground"
     x-data="{
        menuOpen: false,
        page: <?php echo json_encode($page_data); ?>,
        site: <?php echo json_encode($site_info); ?>
     }">

    <!-- Header -->
    <header class="bg-wp-primary text-white py-4">
        <div class="wp-container flex justify-between items-center">
            <h1 class="text-2xl font-bold" x-text="site.name"></h1>

            <!-- Mobile menu toggle -->
            <button class="md:hidden px-3 py-1 border rounded"
                    x-on:click="menuOpen = !menuOpen">
                <span x-text="menuOpen ? 'Close' : 'Menu'"></span>
            </button>

            <!-- Navigation -->
            <nav class="hidden md:block">
                <ul class="flex space-x-4">
                    <li><a href="/" class="hover:underline">Home</a></li>
                    <li><a href="/about" class="hover:underline">About</a></li>
                    <li><a href="/contact" class="hover:underline">Contact</a></li>
                </ul>
            </nav>
        </div>

        <!-- Mobile menu -->
        <div x-show="menuOpen"
             x-transition:enter="transition ease-out duration-200"
             x-transition:enter-start="opacity-0 transform -translate-y-4"
             x-transition:enter-end="opacity-100 transform translate-y-0"
             class="wp-container mt-2 py-3">
            <nav>
                <ul class="space-y-2">
                    <li><a href="/" class="block hover:underline">Home</a></li>
                    <li><a href="/about" class="block hover:underline">About</a></li>
                    <li><a href="/contact" class="block hover:underline">Contact</a></li>
                </ul>
            </nav>
        </div>
    </header>

    <!-- Main content -->
    <main class="wp-container py-wp-40">
        <article class="max-w-3xl mx-auto">
            <h2 class="text-wp-x-large font-bold mb-4" x-text="page.title"></h2>

            <div class="mb-2 text-sm text-gray-600">
                <span>Published on </span>
                <span x-text="page.date"></span>
                <span> by </span>
                <span x-text="page.author"></span>
            </div>

            <!-- Editable region for Gutenberg blocks -->
            <div class="mt-6 prose max-w-none">
                <?php
                if (have_posts()) :
                    while (have_posts()) : the_post();
                        the_content();
                    endwhile;
                endif;
                ?>
            </div>
        </article>
    </main>

    <!-- Footer -->
    <footer class="bg-wp-secondary text-white py-8">
        <div class="wp-container">
            <div class="flex flex-col md:flex-row justify-between">
                <div class="mb-4 md:mb-0">
                    <h3 class="text-lg font-semibold mb-2" x-text="site.name"></h3>
                    <p class="text-sm" x-text="site.description"></p>
                </div>
                <div>
                    <p>&copy; <?php echo date('Y'); ?> <span x-text="site.name"></span></p>
                    <p class="text-sm mt-1">All rights reserved.</p>
                </div>
            </div>
        </div>
    </footer>
</div>

<?php get_footer(); ?>
```

## Server-Side Rendering with Alpine.js and TailwindCSS

### Understanding SSR vs CSR

When building WordPress templates with Alpine.js and TailwindCSS, it's important to understand the difference between Server-Side Rendering (SSR) and Client-Side Rendering (CSR):

**Server-Side Rendering (SSR):**

- Content is generated on the server using PHP
- HTML is delivered to the browser fully formed
- TailwindCSS classes are applied on the server-side
- Search engines can easily index the content
- Pages load with content already visible
- Alpine.js enhances already-rendered content

**Client-Side Rendering (CSR):**

- Content is generated in the browser using JavaScript
- Initial HTML is minimal, with JS fetching/creating content
- Search engines may struggle to index content
- Pages may show loading states before content appears
- Content depends on JavaScript execution

For WordPress sites, SSR with Alpine.js enhancement and TailwindCSS styling provides the best balance of SEO, performance, and interactivity:

```
┌─────────────────────────────┐
│ Server-Side Rendering       │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ WordPress PHP Processes     │
├─────────────────────────────┤
│ 1. Database Queries         │
│ 2. Content Preparation      │
│ 3. HTML Generation          │
│ 4. TailwindCSS Classes      │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Fully-Formed HTML Response  │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Browser Receives Complete   │
│ HTML Document               │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Alpine.js Initialization    │
├─────────────────────────────┤
│ Adds Interactivity to       │
│ Already-Rendered Content    │
└─────────────────────────────┘
```

The key principle for performance and SEO is to use PHP to render all content with TailwindCSS classes and use Alpine.js only for interactivity.

### Template Best Practices

1. **Server-Side Render Everything First**

   - Generate all content via PHP
   - Use WordPress functions like `the_title()`, `the_content()`
   - Apply TailwindCSS utility classes on server-side
   - Avoid Alpine.js directives like `x-for` or `x-html` for main content

2. **Add Interactivity with Alpine.js**
   - Use Alpine.js only for UI interactions
   - Toggle elements, handle clicks, manage simple state
   - Enhance existing server-rendered content

Example of proper SSR with Alpine.js interactivity and TailwindCSS styling:

```php
<!-- FAQ Accordion -->
<div class="accordion max-w-3xl mx-auto my-8 divide-y divide-gray-200" x-data="{ active: null }">
    <?php
    $faq_items = get_field('faq_items'); // Using ACF as an example
    if ($faq_items) :
        foreach ($faq_items as $index => $item) :
    ?>
    <div class="accordion-item py-4">
        <h3 class="accordion-header flex justify-between items-center cursor-pointer text-wp-large font-medium"
            x-on:click="active = active === <?php echo $index; ?> ? null : <?php echo $index; ?>">
            <?php echo esc_html($item['title']); ?>
            <span class="text-wp-primary text-xl" x-text="active === <?php echo $index; ?> ? '−' : '+'"></span>
        </h3>
        <div class="accordion-content mt-2 text-gray-600"
             x-show="active === <?php echo $index; ?>"
             x-transition:enter="transition ease-out duration-200"
             x-transition:enter-start="opacity-0 transform -translate-y-2"
             x-transition:enter-end="opacity-100 transform translate-y-0">
            <?php echo wp_kses_post($item['content']); ?>
        </div>
    </div>
    <?php
        endforeach;
    endif;
    ?>
</div>
```

## Template Conditionals

### Understanding Template Conditionals

In WordPress, **template conditionals** are rules that determine where and when a template should be applied. They allow you to create different layouts for different parts of your website based on the context.

Template conditionals represent the "what" in the question "what should this template apply to?" and are a fundamental part of WordPress's template hierarchy system.

#### Template Conditional Diagram

```
WordPress Request
        │
        ▼
┌───────────────────┐
│ Template Selection│
└─────────┬─────────┘
          │
          ▼
┌───────────────────────────────────────┐
│           Conditional Checks          │
├───────────────────────────────────────┤
│                                       │
│  ┌─────────────┐    ┌─────────────┐   │
│  │ Post Type   │    │ Archive     │   │
│  │ Conditionals│    │ Conditionals│   │
│  │ is_page()   │    │ is_archive()│   │
│  │ is_single() │    │ is_category()   │
│  │ is_singular()    │ is_tag()    │   │
│  └─────────────┘    └─────────────┘   │
│                                       │
│  ┌─────────────┐    ┌─────────────┐   │
│  │ Special Page│    │ Hierarchical│   │
│  │ Conditionals│    │ Conditionals│   │
│  │ is_front_page()  │ is_tax()    │   │
│  │ is_home()   │    │ has_term()  │   │
│  │ is_404()    │    │             │   │
│  └─────────────┘    └─────────────┘   │
└───────────────┬───────────────────────┘
                │
                ▼
┌───────────────────────────────────────┐
│     Custom Template Based on Match    │
└───────────────────────────────────────┘
```

You can apply your custom templates automatically based on WordPress conditional tags:

```php
// Apply your template to specific conditions
add_filter('template_include', 'conditional_template_loader');
function conditional_template_loader($template) {
    // Get current template slug
    $template_slug = get_page_template_slug();

    // If template is explicitly set, use that
    if($template_slug === 'alpine-template.php') {
        return plugin_dir_path(__FILE__) . 'templates/alpine-template.php';
    }

    // Apply template to homepage automatically
    if(is_front_page() && !$template_slug) {
        return plugin_dir_path(__FILE__) . 'templates/home-template.php';
    }

    // Apply template to archive pages
    if(is_archive() && !$template_slug) {
        return plugin_dir_path(__FILE__) . 'templates/archive-template.php';
    }

    return $template;
}
```

### Common Conditional Tags

- `is_front_page()` - Home page
- `is_home()` - Blog posts page
- `is_page()` - Any page
- `is_single()` - Any post
- `is_archive()` - Any archive page
- `is_category()` - Category archive
- `is_tag()` - Tag archive
- `is_author()` - Author archive
- `is_search()` - Search results

### Ensuring Your Templates Take Precedence

When using the `template_include` filter, you might need to ensure your templates take precedence over the active theme or other plugins. By default, WordPress processes filter callbacks in sequence based on their priority (default is 10).

To make your templates "win" over others, use a higher priority number:

```php
// Use a high priority number (e.g., 9999) to ensure your template gets applied last
add_filter('template_include', 'your_conditional_template_loader', 9999);
function your_conditional_template_loader($template) {
    // Your conditional checks
    if (is_front_page()) {
        $new_template = plugin_dir_path(__FILE__) . 'templates/home-template.php';
        if (file_exists($new_template)) {
            return $new_template;
        }
    }

    return $template;
}
```

## Passing WordPress Data to Alpine.js

You can pass data from WordPress to Alpine.js using JSON encoding:

```php
<?php
// Get data from WordPress
$posts = get_posts([
    'numberposts' => 5,
    'post_status' => 'publish'
]);

$posts_data = [];
foreach($posts as $post) {
    $posts_data[] = [
        'id' => $post->ID,
        'title' => $post->post_title,
        'excerpt' => get_the_excerpt($post),
        'permalink' => get_permalink($post->ID),
        'thumbnail' => get_the_post_thumbnail_url($post->ID, 'medium')
    ];
}
?>

<div x-data="{
    posts: <?php echo json_encode($posts_data); ?>,
    activePost: null
}" class="wp-container py-8">
    <h2 class="text-2xl font-bold mb-6">Recent Posts</h2>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <template x-for="post in posts" :key="post.id">
            <div class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow duration-300"
                 x-on:mouseenter="activePost = post.id"
                 x-on:mouseleave="activePost = null">
                <img x-bind:src="post.thumbnail" alt="" x-show="post.thumbnail" class="w-full h-48 object-cover">
                <div class="p-4">
                    <h3 class="text-lg font-semibold mb-2" x-text="post.title"></h3>
                    <p class="text-gray-600 mb-4" x-text="post.excerpt"></p>
                    <a x-bind:href="post.permalink"
                       x-bind:class="{ 'bg-wp-primary': activePost === post.id, 'bg-wp-secondary': activePost !== post.id }"
                       class="inline-block px-4 py-2 text-white rounded transition-colors duration-300">
                       Read More
                    </a>
                </div>
            </div>
        </template>
    </div>
</div>
```

## AJAX Integration

To handle dynamic data loading with Alpine.js and TailwindCSS:

```php
<?php
// Register AJAX endpoint
add_action('wp_ajax_load_more_posts', 'load_more_posts');
add_action('wp_ajax_nopriv_load_more_posts', 'load_more_posts');
function load_more_posts() {
    check_ajax_referer('posts_load_more', 'nonce');

    $page = isset($_POST['page']) ? intval($_POST['page']) : 1;
    $posts_query = new WP_Query([
        'post_type' => 'post',
        'posts_per_page' => 5,
        'paged' => $page,
    ]);

    $posts_data = [];
    if($posts_query->have_posts()) {
        while($posts_query->have_posts()) {
            $posts_query->the_post();
            $posts_data[] = [
                'id' => get_the_ID(),
                'title' => get_the_title(),
                'excerpt' => get_the_excerpt(),
                'permalink' => get_permalink(),
                'thumbnail' => get_the_post_thumbnail_url(get_the_ID(), 'medium')
            ];
        }
        wp_reset_postdata();
    }

    wp_send_json_success([
        'posts' => $posts_data,
        'hasMore' => $page < $posts_query->max_num_pages
    ]);
}

// In your template:
?>
<div x-data="{
    posts: <?php echo json_encode($initial_posts_data); ?>,
    page: 1,
    loading: false,
    hasMore: <?php echo $initial_query->max_num_pages > 1 ? 'true' : 'false'; ?>,
    loadMore() {
        if(this.loading || !this.hasMore) return;

        this.loading = true;
        this.page++;

        const formData = new FormData();
        formData.append('action', 'load_more_posts');
        formData.append('page', this.page);
        formData.append('nonce', '<?php echo wp_create_nonce('posts_load_more'); ?>');

        fetch('<?php echo admin_url('admin-ajax.php'); ?>', {
            method: 'POST',
            body: formData
        })
        .then(response => response.json())
        .then(data => {
            if(data.success) {
                this.posts = [...this.posts, ...data.data.posts];
                this.hasMore = data.data.hasMore;
            }
            this.loading = false;
        })
        .catch(error => {
            console.error('Error loading posts:', error);
            this.loading = false;
        });
    }
}" class="wp-container py-8">
    <h2 class="text-2xl font-bold mb-6">Blog Posts</h2>

    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
        <template x-for="post in posts" :key="post.id">
            <div class="bg-white rounded-lg shadow-md overflow-hidden">
                <img x-bind:src="post.thumbnail" alt="" x-show="post.thumbnail" class="w-full h-48 object-cover">
                <div class="p-4">
                    <h3 class="text-lg font-semibold mb-2" x-text="post.title"></h3>
                    <p class="text-gray-600 mb-4" x-text="post.excerpt"></p>
                    <a x-bind:href="post.permalink" class="wp-button inline-block">
                        Read More
                    </a>
                </div>
            </div>
        </template>
    </div>

    <div class="flex justify-center mt-8">
        <button x-on:click="loadMore()"
                x-bind:disabled="loading || !hasMore"
                x-show="hasMore"
                class="px-6 py-2 bg-wp-primary text-white rounded hover:bg-opacity-90 transition-colors disabled:opacity-50 disabled:cursor-not-allowed">
            <span x-text="loading ? 'Loading...' : 'Load More'"></span>
        </button>
    </div>
</div>
```

## Deployment

### Build Process

Before deploying your plugin, run the production build:

```bash
npm run build
```

This creates optimized assets in the `build/` directory with:

- Minified code
- Content-based hashing for cache busting
- Tree-shaking to remove unused code
- Asset optimization
- Purged TailwindCSS for reduced CSS size

```
┌─────────────────────────────┐
│ Build & Deployment Process  │
└───────────────┬─────────────┘
                │
                ▼
┌─────────────────────────────┐
│ 1. Run Production Build     │
│   npm run build             │
└───────────────┬─────────────┘
                │
                ▼
┌─────────────────────────────┐
│ 2. Optimize Assets          │
│   ✓ Minification           │
│   ✓ Tree-shaking           │
│   ✓ Cache busting          │
│   ✓ TailwindCSS purging    │
└───────────────┬─────────────┘
                │
                ▼
┌─────────────────────────────┐
│ 3. Package Plugin           │
│   Include:                  │
│   - PHP files               │
│   - build/ directory        │
│   - package.json            │
└───────────────┬─────────────┘
                │
                ▼
┌─────────────────────────────┐
│ 4. Deploy to WordPress      │
│   - Upload ZIP              │
│   - Activate plugin         │
└───────────────┬─────────────┘
                │
                ▼
┌─────────────────────────────┐
│ 5. Verify Installation      │
│   - Check templates         │
│   - Test functionality      │
└─────────────────────────────┘
```

Your deployment should include:

- All PHP files (templates, main plugin file)
- The `build/` directory with compiled assets
- Configuration files like `tailwind.config.js`
- `package.json` and `README.md` for documentation

## References

- [WordPress Theme Handbook: Page Templates](https://developer.wordpress.org/themes/template-files-section/page-template-files/)
- [Alpine.js Documentation](https://alpinejs.dev/start-here)
- [TailwindCSS Documentation](https://tailwindcss.com/docs)
- [WordPress Template Conditionals](https://developer.wordpress.org/themes/basics/conditional-tags/)
- [WordPress Scripts Package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/)
- [WordPress Global Styles](https://developer.wordpress.org/block-editor/how-to-guides/themes/theme-json/)
