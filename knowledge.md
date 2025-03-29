# WordPress Plugin with Alpine.js Custom Page Templates

## Overview

This document provides a step-by-step guide for creating WordPress plugins that render Alpine.js-powered pages using custom page templates. This approach allows you to:

1. Create interactive pages using Alpine.js
2. Allow WordPress users to edit certain parts of these pages using Gutenberg blocks
3. Make the solution compatible with any WordPress theme
4. Create a setup that doesn't require modifying or creating a theme
5. Leverage modern build tools like npm and @wordpress/scripts for asset management

## Learning Journey: From Concept to Implementation

This guide takes you on a complete journey through creating a WordPress plugin that seamlessly integrates Alpine.js for interactivity while leveraging WordPress's core strengths. By the end, you'll understand how to build a modern, interactive WordPress site without the complexity of a full JavaScript framework or the limitations of traditional themes.

### What You'll Learn

Your journey begins with understanding the project structure and modern build setup, then progresses through implementing custom blocks, server-side templates, conditional display logic, and finally to advanced techniques like site-wide template overrides.

The concepts build progressively:

1. **Modern Development Environment** - You'll start by setting up a professional build system with npm and @wordpress/scripts, providing the foundation for efficient development.

2. **Custom Block Development** - You'll learn to create custom Gutenberg blocks that integrate with your template system, giving content editors precise control over page content.

3. **Template System** - You'll implement a server-side template system that works with any WordPress theme, maintaining SEO benefits while adding interactivity.

4. **Conditional Logic** - You'll apply templates intelligently using WordPress conditional tags, creating different layouts for different parts of your site automatically.

5. **Alpine.js Integration** - You'll leverage Alpine.js for client-side interactivity without sacrificing server-side rendering or SEO benefits.

6. **Real-World Application** - Finally, you'll explore advanced techniques like site-wide template overrides and Alpine.js stores for global state management.

Each section builds on the previous one, forming a complete system where server-side rendering handles content delivery while Alpine.js adds targeted interactivity exactly where needed.

By completing this guide, you'll have the knowledge to create maintainable, high-performance WordPress sites with modern JavaScript features that work with—not against—WordPress's content management capabilities.

## Table of Contents

1. [Complete Workflow Overview](#complete-workflow-overview)
   - [Implementation Flowchart](#implementation-flowchart)
   - [How the Parts Work Together](#how-the-parts-work-together)
   - [File Structure Overview](#file-structure-overview)
2. [Project Structure](#project-structure)
3. [Build Setup](#build-setup)
   - [NPM Configuration](#npm-configuration)
   - [WordPress Scripts](#wordpress-scripts)
   - [Asset Management](#asset-management)
   - [Development Environment](#development-environment)
4. [Custom Block Development](#custom-block-development)
   - [Block Structure](#block-structure)
   - [Creating a Custom Block](#creating-a-custom-block)
   - [Registering Blocks with WordPress](#registering-blocks-with-wordpress)
   - [Using Custom Blocks in Templates](#using-custom-blocks-in-templates)
   - [Dynamic Blocks vs Static Blocks](#dynamic-blocks-vs-static-blocks)
5. [Backend Implementation](#backend-implementation)
   - [Registering Custom Templates](#registering-custom-templates)
   - [Creating Template Files](#creating-template-files)
   - [Auto-Creating Pages (Optional)](#auto-creating-pages-optional)
6. [Template Conditionals](#template-conditionals)
   - [Understanding Template Conditionals](#understanding-template-conditionals)
   - [Applying Templates to Specific Conditions](#applying-templates-to-specific-conditions)
   - [Common Conditional Patterns](#common-conditional-patterns)
   - [Ensuring Your Templates Take Precedence](#ensuring-your-templates-take-precedence)
7. [Server-Side Rendered Templates](#server-side-rendered-templates)
   - [Overview](#overview)
   - [Implementing SSR with Alpine.js](#implementing-ssr-with-alpinejs)
   - [Applying Static Template Conditionals](#applying-static-template-conditionals)
   - [Server-Side Rendering Best Practices](#server-side-rendering-best-practices)
8. [Alpine.js Implementation](#alpinejs-implementation)
   - [Installing Alpine.js via NPM](#installing-alpinejs-via-npm)
   - [Bundling Alpine.js](#bundling-alpinejs)
   - [Using Alpine.js for Interactivity](#using-alpine.js-for-interactivity)
   - [Passing Limited Data to Alpine.js](#passing-limited-data-to-alpinejs)
   - [Combining WordPress Hooks with Alpine.js](#combining-wordpress-hooks-with-alpinejs)
9. [Plugin Development](#plugin-development)
   - [Main Plugin File](#main-plugin-file)
   - [Enqueuing Built Assets](#enqueuing-built-assets)
   - [Cache Busting](#cache-busting)
10. [Advanced Techniques](#advanced-techniques)
    - [Multiple Editable Regions](#multiple-editable-regions)
    - [Custom Alpine Directives](#custom-alpine-directives)
    - [Alpine.js Stores](#alpinejs-stores)
    - [Site-Wide Header and Footer Override](#site-wide-header-and-footer-override)
11. [Deployment](#deployment)
    - [Build Process](#build-process)
    - [Deployment Strategy](#deployment-strategy)
12. [References](#references)

## Complete Workflow Overview

This section provides a comprehensive overview of how all the components of an Alpine.js WordPress plugin work together, from registration to rendering.

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
│ Enqueue Alpine.js & CSS     │            │ WordPress
└──────────────┬──────────────┘            │ Initialization
               ▼                           │ Process
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
│ 5. get_footer()             │
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
   - It enqueues Alpine.js and CSS using `wp_enqueue_scripts`

2. **Template Selection**
   - When a user visits a page, WordPress checks which template to use
   - If your template is selected (either manually or via conditionals), WordPress runs your template loader
   - Your custom template file is loaded instead of the theme's default template

3. **Server-Side Rendering**
   - Your template prepares data using WordPress functions
   - It renders the HTML structure server-side using PHP
   - It includes Alpine.js directives for interactive elements
   - All content is rendered by the server, with Alpine.js only handling interactivity

4. **Client-Side Interactivity**
   - When the page loads in the browser, Alpine.js initializes
   - It adds interactivity to elements with x-data and other directives
   - The user sees a fully rendered page that becomes interactive without any layout shifts

### File Structure Overview

A complete Alpine.js WordPress plugin implementation with a modern build process typically includes these files:

```
your-plugin/
├── build/                       # Built assets (created by build process)
│   ├── index.js                 # Bundled JavaScript with Alpine.js
│   ├── index.asset.php          # Dependencies and version information
│   └── style.css                # Compiled styles
├── src/                         # Source files for development
│   ├── js/
│   │   ├── index.js             # Main JavaScript entry point
│   │   └── components/          # Alpine.js components
│   │       └── accordion.js     # Example component
│   └── scss/
│       └── style.scss           # SCSS styles
├── templates/                   # Template files
│   ├── alpine-template.php      # Basic custom page template
│   ├── home-template.php        # Home page template
│   └── archive-template.php     # Archive template
├── includes/                    # PHP includes and helpers
│   └── template-functions.php   # Template helper functions
├── your-plugin.php              # Main plugin file
├── package.json                 # NPM dependencies and scripts
└── webpack.config.js            # @wordpress/scripts config overrides (optional)
```

## Project Structure

Here's a detailed file structure for your Alpine.js WordPress plugin with NPM and @wordpress/scripts:

```
your-plugin/
├── build/                       # Built assets (generated via build process)
│   ├── index.js                 # Bundled JavaScript including Alpine.js
│   ├── blocks.js                # Bundled custom blocks
│   ├── index.asset.php          # Dependency and version data for WP
│   ├── blocks.asset.php         # Dependency data for blocks
│   └── style.css                # Compiled CSS
├── src/                         # Source code
│   ├── js/
│   │   ├── index.js             # Main JS entry point
│   │   ├── alpine-init.js       # Alpine.js initialization
│   │   └── components/          # Alpine.js components and directives
│   │       └── accordion.js     # Example component
│   ├── blocks/                  # Custom blocks
│   │   ├── index.js             # Block entry file
│   │   └── featured-content/    # Example block
│   │       ├── index.js         # Block registration
│   │       ├── edit.js          # Editor component
│   │       ├── save.js          # Saved content component
│   │       └── style.scss       # Block styles
│   └── scss/
│       └── style.scss           # Main SCSS file
├── templates/                   # WordPress page templates
│   ├── alpine-template.php      # Basic page template
│   ├── home-template.php        # Home page template
│   └── archive-template.php     # Archive template
├── includes/                    # PHP includes
│   └── template-functions.php   # Template helper functions
├── your-plugin.php              # Main plugin file
├── package.json                 # NPM configuration
├── .gitignore                   # Git ignore file
└── README.md                    # Documentation
```

#### File Responsibilities

| File | Purpose |
|------|---------|
| `your-plugin.php` | Main plugin file: hooks, template registration, built asset enqueuing |
| `src/js/index.js` | JavaScript entry point that imports Alpine.js |
| `build/` | Contains compiled and optimized assets |
| `templates/alpine-template.php` | Main template file selected via page attribute |
| `package.json` | Defines dependencies and build scripts |

## Build Setup

### NPM Configuration

Create a `package.json` file in your plugin root to manage dependencies and scripts:

```json
{
  "name": "your-plugin",
  "version": "1.0.0",
  "description": "WordPress plugin with Alpine.js custom templates",
  "main": "build/index.js",
  "scripts": {
    "build": "wp-scripts build src/js/index.js src/blocks/index.js --output-path=build",
    "start": "wp-scripts start src/js/index.js src/blocks/index.js --output-path=build",
    "check-engines": "wp-scripts check-engines",
    "check-licenses": "wp-scripts check-licenses",
    "format": "wp-scripts format",
    "lint:css": "wp-scripts lint-style",
    "lint:js": "wp-scripts lint-js",
    "packages-update": "wp-scripts packages-update"
  },
  "author": "Your Name",
  "license": "GPL-2.0-or-later",
  "devDependencies": {
    "@wordpress/scripts": "^25.0.0"
  },
  "dependencies": {
    "alpinejs": "^3.12.0"
  }
}
```

### WordPress Scripts

[`@wordpress/scripts`](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/) is an official WordPress package that provides optimized configurations for bundling JavaScript and CSS assets. It includes:

- Webpack and webpack-cli
- Babel for JavaScript transpilation
- PostCSS and SASS for CSS processing
- ESLint and Stylelint for code linting
- Prettier for code formatting

To install all dependencies:

```bash
npm install
```

### Asset Management

Create a JavaScript entry point at `src/js/index.js`:

```javascript
// Import Alpine.js
import Alpine from 'alpinejs';

// Import your custom components/directives
import './components/accordion';

// Make Alpine globally available for debugging
window.Alpine = Alpine;

// Initialize Alpine
document.addEventListener('DOMContentLoaded', () => {
  Alpine.start();
});
```

Create your SCSS styling at `src/scss/style.scss`:

```scss
// Variables
$primary-color: #3490dc;
$secondary-color: #38c172;

// Base styles
.alpine-app {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  
  .site-header {
    background-color: $primary-color;
    color: white;
    padding: 1rem;
  }
  
  .site-content {
    padding: 2rem;
  }
  
  .site-footer {
    background-color: #f8f9fa;
    padding: 1rem;
    text-align: center;
  }
}

// Component styles
@import 'components/accordion';
```

### Development Environment

Start the development environment with:

```bash
npm start
```

This will:
- Watch for file changes
- Automatically rebuild assets when files change
- Generate source maps for debugging

For production builds:

```bash
npm run build
```

This will:
- Optimize and minify all assets
- Generate unique filenames for cache busting
- Create asset dependency files for WordPress

## Custom Block Development

Adding custom blocks to your Alpine.js template plugin extends its functionality, allowing content editors to have more control over their page layouts. The @wordpress/scripts package we're already using makes block development significantly easier.

### Block Structure

A basic custom block has the following components:

1. **Registration script**: Registers the block with WordPress and defines its attributes
2. **Edit component**: The React component that renders in the editor
3. **Save component**: Defines the markup saved to the database and rendered on the frontend
4. **Style files**: CSS/SCSS for styling the block on both editor and frontend

### Creating a Custom Block

Let's create a simple "Featured Content" block that displays content in an attractive format:

1. First, create a blocks entry point at `src/blocks/index.js`:

```javascript
/**
 * Import and register all blocks
 */
import './featured-content';
```

2. Create the block folder structure:

```
src/blocks/featured-content/
├── index.js       # Registration
├── edit.js        # Editor component
├── save.js        # Frontend component
└── style.scss     # Block styles
```

3. Create the block registration file at `src/blocks/featured-content/index.js`:

```javascript
/**
 * WordPress dependencies
 */
import { registerBlockType } from '@wordpress/blocks';

/**
 * Internal dependencies
 */
import Edit from './edit';
import Save from './save';
import './style.scss';

registerBlockType('your-plugin/featured-content', {
    apiVersion: 2,
    title: 'Featured Content',
    description: 'Display content in an attractive featured box',
    category: 'design',
    icon: 'star-filled',
    supports: {
        html: false,
        align: ['wide', 'full'],
    },
    attributes: {
        title: {
            type: 'string',
            default: 'Featured Content',
        },
        content: {
            type: 'string',
            default: 'Add your featured content here',
        },
        backgroundColor: {
            type: 'string',
            default: '#f8f9fa',
        },
        textColor: {
            type: 'string',
            default: '#333333',
        },
    },
    edit: Edit,
    save: Save,
});
```

4. Create the editor component at `src/blocks/featured-content/edit.js`:

```javascript
/**
 * WordPress dependencies
 */
import { __ } from '@wordpress/i18n';
import { 
    useBlockProps, 
    RichText, 
    InspectorControls, 
    PanelColorSettings 
} from '@wordpress/block-editor';
import { 
    Panel, 
    PanelBody 
} from '@wordpress/components';

const Edit = ({ attributes, setAttributes }) => {
    const { title, content, backgroundColor, textColor } = attributes;
    const blockProps = useBlockProps({
        style: {
            backgroundColor,
            color: textColor,
            padding: '20px',
            borderRadius: '5px',
        }
    });

    return (
        <>
            <InspectorControls>
                <Panel>
                    <PanelBody title={__('Color Settings', 'your-plugin')}>
                        <PanelColorSettings
                            title={__('Color Settings', 'your-plugin')}
                            colorSettings={[
                                {
                                    value: backgroundColor,
                                    onChange: (value) => setAttributes({ backgroundColor: value }),
                                    label: __('Background Color', 'your-plugin'),
                                },
                                {
                                    value: textColor,
                                    onChange: (value) => setAttributes({ textColor: value }),
                                    label: __('Text Color', 'your-plugin'),
                                },
                            ]}
                        />
                    </PanelBody>
                </Panel>
            </InspectorControls>

            <div {...blockProps}>
                <RichText
                    tagName="h3"
                    value={title}
                    onChange={(value) => setAttributes({ title: value })}
                    placeholder={__('Featured Title', 'your-plugin')}
                    className="featured-content-title"
                />
                <RichText
                    tagName="div"
                    value={content}
                    onChange={(value) => setAttributes({ content: value })}
                    placeholder={__('Featured content goes here...', 'your-plugin')}
                    className="featured-content-body"
                />
            </div>
        </>
    );
};

export default Edit;
```

5. Create the save component at `src/blocks/featured-content/save.js`:

```javascript
/**
 * WordPress dependencies
 */
import { useBlockProps, RichText } from '@wordpress/block-editor';

const Save = ({ attributes }) => {
    const { title, content, backgroundColor, textColor } = attributes;
    const blockProps = useBlockProps.save({
        style: {
            backgroundColor,
            color: textColor,
            padding: '20px',
            borderRadius: '5px',
        }
    });

    return (
        <div {...blockProps}>
            <RichText.Content
                tagName="h3"
                value={title}
                className="featured-content-title"
            />
            <RichText.Content
                tagName="div"
                value={content}
                className="featured-content-body"
            />
        </div>
    );
};

export default Save;
```

6. Add styles to `src/blocks/featured-content/style.scss`:

```scss
.wp-block-your-plugin-featured-content {
    margin: 2rem 0;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    transition: box-shadow 0.3s ease;

    &:hover {
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
    }

    .featured-content-title {
        margin-top: 0;
        margin-bottom: 1rem;
        font-size: 1.5rem;
    }

    .featured-content-body {
        line-height: 1.6;
    }
}
```

### Registering Blocks with WordPress

Now that we've created our block files, we need to:

1. Add a block entry point to webpack
2. Register the blocks with WordPress

First, let's modify our `package.json` to add a new entry point:

```json
{
  "scripts": {
    "build": "wp-scripts build src/js/index.js src/blocks/index.js --output-path=build",
    "start": "wp-scripts start src/js/index.js src/blocks/index.js --output-path=build",
    // other scripts remain the same
  }
}
```

Then, update the main plugin file to register and enqueue the blocks:

```php
/**
 * Register and enqueue block assets
 */
function register_custom_blocks() {
    // Skip block registration if Gutenberg is not available
    if (!function_exists('register_block_type')) {
        return;
    }

    // Register block script
    $asset_file = include plugin_dir_path(__FILE__) . 'build/blocks.asset.php';
    
    wp_register_script(
        'your-plugin-blocks',
        plugin_dir_url(__FILE__) . 'build/blocks.js',
        $asset_file['dependencies'],
        $asset_file['version'],
        true
    );

    // Register block styles
    wp_register_style(
        'your-plugin-blocks',
        plugin_dir_url(__FILE__) . 'build/style.css',
        array(),
        $asset_file['version']
    );

    // Register each block
    register_block_type('your-plugin/featured-content', array(
        'editor_script' => 'your-plugin-blocks',
        'editor_style'  => 'your-plugin-blocks',
        'style'         => 'your-plugin-blocks',
    ));
}
add_action('init', 'register_custom_blocks');
```

### Using Custom Blocks in Templates

You can use your custom blocks in two ways:

1. **In the block editor**: Your blocks will appear in the inserter alongside standard blocks
2. **In template files**: You can include them directly in your template HTML files

To include a custom block in your template file, add it like this:

```html
<!-- wp:your-plugin/featured-content {"title":"Welcome to Our Site","backgroundColor":"#e9f7fe","textColor":"#0a5082"} -->
<div class="wp-block-your-plugin-featured-content" style="background-color:#e9f7fe;color:#0a5082;padding:20px;border-radius:5px">
    <h3 class="featured-content-title">Welcome to Our Site</h3>
    <div class="featured-content-body">This is a custom featured content block that showcases important information.</div>
</div>
<!-- /wp:your-plugin/featured-content -->
```

### Dynamic Blocks vs Static Blocks

There are two types of blocks you can create:

1. **Static blocks**: The block we created above is a static block. Its content is saved in the post content as HTML and rendered exactly as saved.

2. **Dynamic blocks**: These are rendered server-side with PHP each time the page loads. They're perfect for content that changes over time or needs to pull from external sources.

Here's how to create a dynamic block:

1. Create the block registration and edit component as before.

2. Instead of a save component, return null:

```javascript
// In your block's index.js
save: () => null,
```

3. Create a server-side render callback in your plugin:

```php
/**
 * Server-side rendering for dynamic blocks
 */
function render_dynamic_featured_block($attributes, $content) {
    $title = isset($attributes['title']) ? $attributes['title'] : 'Featured Content';
    $content = isset($attributes['content']) ? $attributes['content'] : '';
    $bg_color = isset($attributes['backgroundColor']) ? $attributes['backgroundColor'] : '#f8f9fa';
    $text_color = isset($attributes['textColor']) ? $attributes['textColor'] : '#333333';
    
    // You can pull dynamic content here, for example:
    // $recent_posts = get_posts(['numberposts' => 3]);
    
    ob_start();
    ?>
    <div class="wp-block-your-plugin-featured-content" 
         style="background-color:<?php echo esc_attr($bg_color); ?>;
                color:<?php echo esc_attr($text_color); ?>;
                padding:20px;
                border-radius:5px;">
        <h3 class="featured-content-title"><?php echo esc_html($title); ?></h3>
        <div class="featured-content-body">
            <?php echo wp_kses_post($content); ?>
            
            <?php // Example of adding dynamic content
            // if (!empty($recent_posts)) {
            //     echo '<ul>';
            //     foreach ($recent_posts as $post) {
            //         echo '<li><a href="' . get_permalink($post) . '">' . $post->post_title . '</a></li>';
            //     }
            //     echo '</ul>';
            // }
            ?>
        </div>
    </div>
    <?php
    return ob_get_clean();
}

/**
 * Register dynamic block
 */
register_block_type('your-plugin/dynamic-featured-content', array(
    'editor_script' => 'your-plugin-blocks',
    'editor_style'  => 'your-plugin-blocks',
    'style'         => 'your-plugin-blocks',
    'render_callback' => 'render_dynamic_featured_block',
    'attributes' => array(
        'title' => array(
            'type' => 'string',
            'default' => 'Featured Content',
        ),
        'content' => array(
            'type' => 'string',
            'default' => '',
        ),
        'backgroundColor' => array(
            'type' => 'string',
            'default' => '#f8f9fa',
        ),
        'textColor' => array(
            'type' => 'string',
            'default' => '#333333',
        ),
    ),
));
```

Dynamic blocks are particularly useful in custom page templates when you need to pull in fresh content each time the page loads.

## Backend Implementation

### Registering Custom Templates

Add this code to your main plugin file to register a custom page template:

```php
/**
 * Register custom page templates
 */
add_filter('theme_page_templates', 'add_alpine_page_template');
function add_alpine_page_template($templates) {
    // Add your custom template(s)
    $templates['alpine-template.php'] = 'Alpine.js Application';
    return $templates;
}

/**
 * Load custom template from plugin directory when selected
 */
add_filter('template_include', 'load_alpine_page_template');
function load_alpine_page_template($template) {
    // Get current page template
    $template_slug = get_page_template_slug();
    
    // Check if our template is selected
    if($template_slug === 'alpine-template.php') {
        // Provide path to the template file in our plugin
        $template = plugin_dir_path(__FILE__) . 'templates/alpine-template.php';
    }
    
    return $template;
}
```

### Creating Template Files

Create `templates/alpine-template.php`:

```php
<?php
/**
 * Template Name: Alpine.js Application
 * Description: A custom template for Alpine.js applications
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

// Get menu data
$menu_locations = get_nav_menu_locations();
$menu_data = array();
if (isset($menu_locations['primary'])) {
    $menu = wp_get_nav_menu_object($menu_locations['primary']);
    $menu_items = wp_get_nav_menu_items($menu->term_id);
    
    foreach ($menu_items as $item) {
        $menu_data[] = array(
            'id' => $item->ID,
            'title' => $item->title,
            'url' => $item->url,
            'parent' => $item->menu_item_parent
        );
    }
}

// Site info
$site_info = array(
    'name' => get_bloginfo('name'),
    'description' => get_bloginfo('description'),
    'url' => home_url()
);
?>

<!-- Alpine.js Application -->
<div class="alpine-app" 
     x-data="{
        menuOpen: false,
        page: <?php echo json_encode($page_data); ?>,
        menu: <?php echo json_encode($menu_data); ?>,
        site: <?php echo json_encode($site_info); ?>
     }">
    
    <!-- Header -->
    <header class="site-header">
        <div class="header-container">
            <h1 class="site-title" x-text="site.name"></h1>
            
            <!-- Mobile menu toggle -->
            <button class="menu-toggle" 
                    x-on:click="menuOpen = !menuOpen">
                <span x-text="menuOpen ? 'Close' : 'Menu'"></span>
            </button>
        </div>
        
        <!-- Navigation -->
        <nav class="site-navigation" 
             x-bind:class="{ 'is-open': menuOpen }">
            <ul class="menu">
                <template x-for="item in menu" :key="item.id">
                    <li class="menu-item">
                        <a x-bind:href="item.url" x-text="item.title"></a>
                    </li>
                </template>
            </ul>
        </nav>
    </header>
    
    <!-- Main content -->
    <main class="site-content">
        <article class="page">
            <h2 class="page-title" x-text="page.title"></h2>
            
            <!-- Editable region for Gutenberg blocks -->
            <div class="page-content">
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
    <footer class="site-footer">
        <p>&copy; <?php echo date('Y'); ?> <span x-text="site.name"></span></p>
    </footer>
</div>

<?php get_footer(); ?>
```

### Auto-Creating Pages (Optional)

If you want to automatically create a page using your template:

```php
/**
 * Create a page with the Alpine.js template on plugin activation
 */
register_activation_hook(__FILE__, 'create_alpine_page');
function create_alpine_page() {
    // Check if page already exists
    $page_check = get_page_by_path('alpine-app');
    
    // Only create page if it doesn't exist
    if (!$page_check) {
        $page = array(
            'post_title'    => 'Alpine.js Application',
            'post_name'     => 'alpine-app',
            'post_status'   => 'publish',
            'post_type'     => 'page',
            'comment_status' => 'closed',
            'ping_status'   => 'closed',
        );
        
        // Insert the page
        $page_id = wp_insert_post($page);
        
        // Set the page template
        if ($page_id) {
            update_post_meta($page_id, '_wp_page_template', 'alpine-template.php');
        }
    }
}
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

Common types of template conditionals include:

1. **Post Type Conditionals** - Apply to specific content types
   - Pages (`is_page()`)
   - Posts (`is_single()`)
   - Custom Post Types (`is_singular('product')`)

2. **Archive Conditionals** - Apply to listing/collection pages
   - Category archives (`is_category()`)
   - Tag archives (`is_tag()`)
   - Author archives (`is_author()`)
   - Date archives (`is_date()`)
   - Search results (`is_search()`)

3. **Special Page Conditionals**
   - Home page (`is_front_page()`)
   - Blog page (`is_home()`)
   - 404 pages (`is_404()`)

4. **Hierarchical Conditionals**
   - Parent/child relationships (`is_page_template()`)
   - Taxonomy hierarchies (`is_tax()`)

### Applying Templates to Specific Conditions

There are two main approaches to apply templates to specific conditionals:

#### 1. Using Template Files with Conditional Logic

This method uses WordPress's standard template file approach but adds conditional logic inside your template loader:

```php
add_filter('template_include', 'conditional_template_loader');
function conditional_template_loader($template) {
    // Custom Product template for specific category
    if (is_singular('product') && has_term('featured', 'product_category')) {
        $new_template = plugin_dir_path(__FILE__) . 'templates/featured-product.php';
        if (file_exists($new_template)) {
            return $new_template;
        }
    }
    
    // Custom template for author archives
    if (is_author()) {
        $new_template = plugin_dir_path(__FILE__) . 'templates/author-profile.php';
        if (file_exists($new_template)) {
            return $new_template;
        }
    }
    
    // Fall back to the default template
    return $template;
}
```

#### 2. Using Template Parts with Conditions

For more granular control, you can conditionally load different template parts within your main template:

```php
// In your template file
get_header();

// Conditionally load different content areas
if (is_single()) {
    // Load single post template part
    include plugin_dir_path(__FILE__) . 'template-parts/content-single.php';
} elseif (is_page()) {
    // Load page template part
    include plugin_dir_path(__FILE__) . 'template-parts/content-page.php';
} elseif (is_archive()) {
    // Load archive template part
    include plugin_dir_path(__FILE__) . 'template-parts/content-archive.php';
}

get_footer();
```

#### 3. Using Meta Options

You can also use post meta to determine which template to use for specific content:

```php
add_filter('template_include', 'meta_based_template_loader');
function meta_based_template_loader($template) {
    if (is_singular()) {
        $custom_template = get_post_meta(get_the_ID(), '_custom_template', true);
        if (!empty($custom_template)) {
            $new_template = plugin_dir_path(__FILE__) . 'templates/' . $custom_template . '.php';
            if (file_exists($new_template)) {
                return $new_template;
            }
        }
    }
    return $template;
}
```

### Common Conditional Patterns

Here are some practical examples of template conditionals for common use cases:

#### Product Single Page Template

```php
add_filter('template_include', 'product_template_loader');
function product_template_loader($template) {
    if (is_singular('product')) {
        return plugin_dir_path(__FILE__) . 'templates/single-product.php';
    }
    return $template;
}
```

#### Category Archive Template

```php
add_filter('template_include', 'category_template_loader');
function category_template_loader($template) {
    if (is_category('news')) {
        return plugin_dir_path(__FILE__) . 'templates/category-news.php';
    }
    return $template;
}
```

#### Template by URL Pattern

```php
add_filter('template_include', 'url_based_template_loader');
function url_based_template_loader($template) {
    // Get current URL path
    $current_url = $_SERVER['REQUEST_URI'];
    
    // Apply template to a specific section of the site
    if (strpos($current_url, '/members/') !== false) {
        return plugin_dir_path(__FILE__) . 'templates/members-area.php';
    }
    
    return $template;
}
```

#### Advanced Multi-Conditional Template

```php
add_filter('template_include', 'advanced_conditional_template');
function advanced_conditional_template($template) {
    // Portfolio items with the "featured" tag
    if (is_singular('portfolio') && has_term('featured', 'portfolio_tag')) {
        return plugin_dir_path(__FILE__) . 'templates/featured-portfolio.php';
    }
    
    // All product category archives except "accessories"
    if (is_tax('product_cat') && !is_tax('product_cat', 'accessories')) {
        return plugin_dir_path(__FILE__) . 'templates/product-category.php';
    }
    
    // Custom author template for specific authors
    if (is_author(array(1, 5, 9))) {  // Author IDs 1, 5, and 9
        return plugin_dir_path(__FILE__) . 'templates/featured-author.php';
    }
    
    return $template;
}
```

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

For cases where you absolutely must ensure your template is applied regardless of other plugins, you can implement a two-step approach with an extremely high priority:

```php
// Set an extreme priority to override anything else
add_filter('template_include', 'force_your_template', 99999);
function force_your_template($template) {
    // Check for a global flag indicating we want to force our template
    global $is_forcing_template;
    
    if ($is_forcing_template && file_exists($is_forcing_template)) {
        return $is_forcing_template;
    }
    
    return $template;
}

// Your main template filter at a moderate priority
add_filter('template_include', 'your_conditional_template_loader', 100);
function your_conditional_template_loader($template) {
    global $is_forcing_template;
    
    if (is_front_page()) {
        $new_template = plugin_dir_path(__FILE__) . 'templates/home-template.php';
        if (file_exists($new_template)) {
            // Set the global variable to force this template
            $is_forcing_template = $new_template;
            return $new_template;
        }
    }
    
    return $template;
}
```

This approach ensures your template will take precedence even if another plugin attempts to override it with a higher priority callback.

## Server-Side Rendered Templates

### Overview

When working with Alpine.js in WordPress, it's important to ensure all content is server-side rendered (SSR) rather than client-side rendered (CSR). This provides better SEO, accessibility, and initial page load performance.

The key principle is to use PHP to render all content and use Alpine.js only for interactivity.

### Implementing SSR with Alpine.js

Here are two complete examples of server-side rendered templates that use Alpine.js only for interactivity:

#### 1. Home Page Template (SSR)

Create `templates/home-template.php`:

```php
<?php
/**
 * Custom Home Page Template
 */
get_header();

// Get featured posts
$featured_posts = get_posts(array(
    'posts_per_page' => 3,
    'post_status' => 'publish'
));
?>

<div class="home-template" x-data="{ menuOpen: false }">
    <!-- Hero Section -->
    <section class="hero">
        <h1><?php echo esc_html(get_bloginfo('name')); ?></h1>
        <p><?php echo esc_html(get_bloginfo('description')); ?></p>
    </section>
    
    <!-- Featured Posts - Server-side rendered -->
    <section class="featured-posts">
        <h2>Featured Posts</h2>
        <div class="posts-grid">
            <?php foreach ($featured_posts as $post): setup_postdata($post); ?>
                <article class="post-card">
                    <?php if (has_post_thumbnail($post->ID)): ?>
                    <div class="post-image">
                        <?php echo get_the_post_thumbnail($post->ID, 'large'); ?>
                    </div>
                    <?php endif; ?>
                    <h3><?php echo esc_html($post->post_title); ?></h3>
                    <div><?php echo get_the_excerpt($post); ?></div>
                    <a href="<?php echo esc_url(get_permalink($post->ID)); ?>">Read More</a>
                </article>
            <?php endforeach; wp_reset_postdata(); ?>
        </div>
    </section>
    
    <!-- Mobile Menu Toggle - Client-side interactivity only -->
    <button class="menu-toggle" 
            x-on:click="menuOpen = !menuOpen">
        <span x-text="menuOpen ? 'Close Menu' : 'Open Menu'"></span>
    </button>
    
    <nav class="navigation" x-bind:class="{ 'is-open': menuOpen }">
        <?php 
        // Output the menu server-side
        wp_nav_menu(array(
            'theme_location' => 'primary',
            'container' => false,
            'menu_class' => 'main-menu'
        )); 
        ?>
    </nav>
    
    <!-- Main Content -->
    <section class="home-content">
        <?php 
        if (have_posts()) : 
            while (have_posts()) : the_post();
                the_content();
            endwhile;
        endif; 
        ?>
    </section>
</div>

<?php get_footer(); ?>
```

#### 2. Archive Template (SSR)

Create `templates/archive-template.php`:

```php
<?php
/**
 * Custom Archive Template
 */
get_header();
?>

<div class="archive-template" x-data="{ filterOpen: false }">
    <!-- Archive Header -->
    <header class="archive-header">
        <h1><?php the_archive_title(); ?></h1>
        <?php if (get_the_archive_description()): ?>
            <div class="archive-description"><?php the_archive_description(); ?></div>
        <?php endif; ?>
    </header>
    
    <!-- Filter Toggle - Client-side interactivity only -->
    <button class="filter-toggle" 
            x-on:click="filterOpen = !filterOpen">
        <span x-text="filterOpen ? 'Hide Filters' : 'Show Filters'"></span>
    </button>
    
    <!-- Filter Sidebar - Uses Alpine only for visibility toggle -->
    <aside class="filters" x-bind:class="{ 'is-open': filterOpen }">
        <?php if (is_active_sidebar('archive-sidebar')): ?>
            <?php dynamic_sidebar('archive-sidebar'); ?>
        <?php endif; ?>
    </aside>
    
    <!-- Posts Listing - Server-side rendered -->
    <div class="posts-listing">
        <?php if (have_posts()): ?>
            <?php while (have_posts()): the_post(); ?>
                <article class="post-item">
                    <?php if (has_post_thumbnail()): ?>
                    <div class="post-image">
                        <?php the_post_thumbnail('medium'); ?>
                    </div>
                    <?php endif; ?>
                    <div class="post-content">
                        <h2><?php the_title(); ?></h2>
                        <div class="post-meta">
                            <span><?php the_date(); ?></span> by 
                            <span><?php the_author(); ?></span>
                        </div>
                        <div><?php the_excerpt(); ?></div>
                        <a href="<?php the_permalink(); ?>" class="read-more">Read More</a>
                    </div>
                </article>
            <?php endwhile; ?>
        <?php else: ?>
            <p>No posts found.</p>
        <?php endif; ?>
    </div>
    
    <!-- Pagination -->
    <div class="pagination">
        <?php echo paginate_links(); ?>
    </div>
</div>

<?php get_footer(); ?>
```

### Applying Static Template Conditionals

You can apply these templates to specific conditions without needing admin configuration. Add this code to your main plugin file:

```php
/**
 * Apply custom templates based on specific conditions
 */
add_filter('template_include', 'apply_custom_templates');
function apply_custom_templates($template) {
    // 1. Custom Home Page Template
    if (is_front_page()) {
        $home_template = plugin_dir_path(__FILE__) . 'templates/home-template.php';
        if (file_exists($home_template)) {
            return $home_template;
        }
    }
    
    // 2. Custom Archive Template (for all post listings)
    if (is_archive() || is_home()) {  // is_home() is for blog posts page
        $archive_template = plugin_dir_path(__FILE__) . 'templates/archive-template.php';
        if (file_exists($archive_template)) {
            return $archive_template;
        }
    }
    
    // For all other cases, let WordPress use its default template
    return $template;
}
```

### Server-Side Rendering Best Practices

To ensure you're properly implementing server-side rendering with Alpine.js:

1. **Avoid Alpine.js directives for content rendering:**
   - **Don't use:** `<template x-for="...">` or `x-html` for main content
   - **Do use:** PHP loops like `foreach` and `while` for content iteration
   
2. **Use Alpine.js only for interactivity:**
   - Toggling UI elements
   - Simple animations
   - Form validation
   - Class binding

3. **Properly escape all outputs:**
   - Use `esc_html()` for text
   - Use `esc_attr()` for attributes
   - Use `esc_url()` for links

4. **Use WordPress template tags:**
   - `the_title()` instead of `x-text="post.title"`
   - `the_content()` instead of `x-html="post.content"`
   - `the_permalink()` instead of `x-bind:href="post.url"`

## Alpine.js Implementation

When implementing Alpine.js with WordPress, we want to ensure proper server-side rendering (SSR) while using Alpine.js only for interactivity.

### Installing Alpine.js via NPM

Instead of loading Alpine.js from a CDN, we'll install it as an NPM package:

```bash
npm install alpinejs
```

This approach provides several advantages:
- Version control over the dependency
- Ability to bundle only what you need
- Better performance when combined with code splitting
- Compatibility with modern build tools

### Bundling Alpine.js

Once Alpine.js is installed, import it in your main JavaScript file (`src/js/index.js`):

```javascript
import Alpine from 'alpinejs';

// Optional: make Alpine available globally for debugging
window.Alpine = Alpine;

// Register custom directives
Alpine.directive('tooltip', (el, { expression }, { evaluate }) => {
  const content = evaluate(expression);
  if (content) {
    // Apply WordPress formatting
    el.innerHTML = content.replace(/\n/g, '<br>');
  }
});

// Initialize Alpine when the DOM is loaded
document.addEventListener('DOMContentLoaded', () => {
  Alpine.start();
});
```

You can also create modular components in separate files:

File: `src/js/components/accordion.js`
```javascript
import Alpine from 'alpinejs';

Alpine.data('accordion', () => ({
  open: false,
  
  toggle() {
    this.open = !this.open;
  }
}));
```

Then import your components in your main file:
```javascript
import Alpine from 'alpinejs';
import './components/accordion';

window.Alpine = Alpine;
Alpine.start();
```

### Using Alpine.js for Interactivity (Not Content Rendering)

Here's an example of a proper use of Alpine.js for interactivity only:

```html
<!-- Accordion - Server-rendered content with Alpine.js interactivity -->
<div class="accordion" x-data="{ active: null }">
    <?php 
    // Server-side rendered accordion items
    $faq_items = get_field('faq_items'); // Using ACF as an example
    if ($faq_items) :
        foreach ($faq_items as $index => $item) : 
    ?>
        <div class="accordion-item">
            <h3 class="accordion-header" 
                x-on:click="active = active === <?php echo $index; ?> ? null : <?php echo $index; ?>">
                <?php echo esc_html($item['title']); ?>
                <span class="icon" x-text="active === <?php echo $index; ?> ? '−' : '+'"></span>
            </h3>
            <div class="accordion-content" 
                 x-show="active === <?php echo $index; ?>"
                 x-transition>
                <?php echo wp_kses_post($item['content']); ?>
            </div>
        </div>
    <?php 
        endforeach;
    endif; 
    ?>
</div>
```

### Passing Limited Data to Alpine.js

While we want to avoid passing large amounts of data to Alpine.js for rendering, small pieces of state data are acceptable:

```php
<?php
// Minimal data for Alpine.js state
$state_data = array(
    'isLoggedIn' => is_user_logged_in(),
    'currentPostId' => get_the_ID(),
    'ajaxUrl' => admin_url('admin-ajax.php'),
    'nonce' => wp_create_nonce('my_ajax_nonce')
);
?>

<div x-data="{ 
    state: <?php echo json_encode($state_data); ?>,
    showLoginForm: false 
}">
    <!-- Login/Logout toggle button -->
    <?php if (is_user_logged_in()) : ?>
        <a href="<?php echo wp_logout_url(get_permalink()); ?>">Log Out</a>
    <?php else : ?>
        <button x-on:click="showLoginForm = !showLoginForm">
            <span x-text="showLoginForm ? 'Cancel' : 'Log In'"></span>
        </button>
    <?php endif; ?>
    
    <!-- Login form toggle with Alpine.js -->
    <div class="login-form" x-show="showLoginForm" x-transition>
        <?php wp_login_form(); ?>
    </div>
</div>
```

### Combining WordPress Hooks with Alpine.js

You can use WordPress hooks to enhance your Alpine.js functionality:

```php
<?php
// Add AJAX support for a feature
add_action('wp_ajax_like_post', 'handle_like_post');
add_action('wp_ajax_nopriv_like_post', 'handle_like_post');
function handle_like_post() {
    // Check nonce
    check_ajax_referer('post_like_nonce', 'nonce');
    
    // Get post ID
    $post_id = isset($_POST['post_id']) ? intval($_POST['post_id']) : 0;
    
    // Process like (demo only)
    $likes = get_post_meta($post_id, 'post_likes', true) ?: 0;
    $likes++;
    update_post_meta($post_id, 'post_likes', $likes);
    
    wp_send_json_success(['likes' => $likes]);
}
?>

<!-- In your template -->
<div class="post-likes" x-data="{ 
    likes: <?php echo intval(get_post_meta(get_the_ID(), 'post_likes', true) ?: 0); ?>,
    isLiked: false,
    likePost() {
        if (this.isLiked) return;
        
        // Create FormData
        const formData = new FormData();
        formData.append('action', 'like_post');
        formData.append('post_id', <?php echo get_the_ID(); ?>);
        formData.append('nonce', '<?php echo wp_create_nonce('post_like_nonce'); ?>');
        
        // Send AJAX request
        fetch('<?php echo admin_url('admin-ajax.php'); ?>', {
            method: 'POST',
            body: formData,
            credentials: 'same-origin'
        })
        .then(response => response.json())
        .then(data => {
            if (data.success) {
                this.likes = data.data.likes;
                this.isLiked = true;
            }
        });
    }
}">
    <button class="like-button" 
            x-on:click="likePost()" 
            x-bind:disabled="isLiked"
            x-bind:class="{ 'is-liked': isLiked }">
        Like this post
    </button>
    <span class="like-count">
        <span x-text="likes"></span> <?php echo _n('like', 'likes', get_post_meta(get_the_ID(), 'post_likes', true) ?: 0, 'text-domain'); ?>
    </span>
</div>
```

### Multiple Editable Regions with SSR

To create multiple editable regions while maintaining server-side rendering:

1. **Create separate post meta fields** for different content sections
2. Use custom Gutenberg blocks to edit these sections
3. In your template, retrieve and display each section:

```php
// In your template file
$header_content = get_post_meta(get_the_ID(), 'header_region', true);
$sidebar_content = get_post_meta(get_the_ID(), 'sidebar_region', true);

// Add to your Alpine.js data
$page_data['regions'] = array(
    'header' => $header_content,
    'sidebar' => $sidebar_content
);
```

Then in your template:

```html
<div class="header-region" x-html="page.regions.header"></div>
<div class="sidebar-region" x-html="page.regions.sidebar"></div>
```

### Custom Alpine Directives

You can register custom Alpine.js directives for common functionality:

```html
<script>
    document.addEventListener('alpine:init', () => {
        // Custom directive for WordPress formatting
        Alpine.directive('wpformat', (el, { expression }, { evaluate }) => {
            const content = evaluate(expression);
            if (content) {
                // Apply WordPress formatting
                el.innerHTML = content.replace(/\n/g, '<br>');
            }
        });
    });
</script>

<!-- Usage -->
<div x-wpformat="page.content"></div>
```

### Alpine.js Stores

For global state management across components:

```html
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.store('userData', {
            isLoggedIn: <?php echo is_user_logged_in() ? 'true' : 'false'; ?>,
            user: <?php echo json_encode(wp_get_current_user()); ?>
        });
    });
</script>

<!-- Usage anywhere in your template -->
<div x-data>
    <template x-if="$store.userData.isLoggedIn">
        <p>Welcome, <span x-text="$store.userData.user.display_name"></span>!</p>
    </template>
    <template x-if="!$store.userData.isLoggedIn">
        <p>Please log in to access all features.</p>
    </template>
</div>
```

### Site-Wide Header and Footer Override

If you want to completely override the theme's header and footer across your site, you can create a plugin that takes control of these elements regardless of the active theme.

#### File Structure for Site-Wide Header and Footer Override

```
your-plugin/
├── your-plugin.php              # Main plugin file with override logic
├── templates/
│   ├── custom-header.php        # Custom header replacement
│   ├── custom-footer.php        # Custom footer replacement
│   ├── home-template.php        # Home page template
│   └── archive-template.php     # Archive page template
└── assets/
    └── css/
        └── custom-style.css     # Custom styles for header/footer
```

#### Plugin-Based Template Override with Template Hierarchy

This approach allows you to override headers and footers site-wide while providing custom templates for specific pages:

```php
/**
 * Template Override System
 * 
 * Provides a way to override headers, footers, and specific page templates
 * regardless of the active theme.
 */

// Override specific templates based on conditions
add_filter('template_include', 'override_templates', 9999);
function override_templates($template) {
    // For home page, use your custom template
    if (is_front_page()) {
        $custom_template = plugin_dir_path(__FILE__) . 'templates/home-template.php';
        if (file_exists($custom_template)) {
            return $custom_template;
        }
    }
    
    // For archive pages, use a custom archive template
    if (is_archive() || is_home()) {
        $custom_template = plugin_dir_path(__FILE__) . 'templates/archive-template.php';
        if (file_exists($custom_template)) {
            return $custom_template;
        }
    }
    
    // For all other pages, return the original template
    // The header/footer will still be overridden below
    return $template;
}

// Override header site-wide
add_action('get_header', 'override_header', 9999);
function override_header($name) {
    // Prevent infinite loop
    remove_action('get_header', 'override_header', 9999);
    
    // Load your custom header instead
    $path = plugin_dir_path(__FILE__) . 'templates/custom-header.php';
    if (file_exists($path)) {
        include $path;
        // Exit to prevent the theme's header from loading
        $GLOBALS['wp_did_header'] = true; 
        return;
    }
    
    // If we got here, reattach our filter for future calls
    add_action('get_header', 'override_header', 9999);
}

// Override footer site-wide
add_action('get_footer', 'override_footer', 9999);
function override_footer($name) {
    // Similar approach to the header
    remove_action('get_footer', 'override_footer', 9999);
    
    $path = plugin_dir_path(__FILE__) . 'templates/custom-footer.php';
    if (file_exists($path)) {
        include $path;
        return;
    }
    
    add_action('get_footer', 'override_footer', 9999);
}
```

#### Header and Footer Override Flowchart

```
┌───────────────────────┐
│ WordPress Loads Theme │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐         ┌───────────────────────┐
│ Theme Calls get_header │─────────► Your Override Action  │
└───────────────────────┘         │ (Priority 9999)       │
                                   └───────────┬───────────┘
                                               │
┌───────────────────────┐                      │
│ Your Custom Header    │◄─────────────────────┘
│ (templates/custom-    │
│  header.php)          │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐         ┌───────────────────────┐
│ Theme Calls get_footer │─────────► Your Override Action  │
└───────────────────────┘         │ (Priority 9999)       │
                                   └───────────┬───────────┘
                                               │
┌───────────────────────┐                      │
│ Your Custom Footer    │◄─────────────────────┘
│ (templates/custom-    │
│  footer.php)          │
└───────────────────────┘
```

With this approach, you need to create these template files in your plugin:

1. **`templates/custom-header.php`**: Your site-wide header replacement
2. **`templates/custom-footer.php`**: Your site-wide footer replacement
3. **`templates/home-template.php`**: Complete template for the home page
4. **`templates/archive-template.php`**: Template for archive/blog pages

## Plugin Development

### Main Plugin File

Create `your-plugin.php` with proper asset enqueuing:

```php
<?php
/**
 * Plugin Name: Alpine.js Template
 * Description: A plugin that integrates Alpine.js with WordPress using custom page templates
 * Version: 1.0.0
 * Author: Your Name
 * Text Domain: alpine-template
 */

// Exit if accessed directly
if (!defined('ABSPATH')) {
    exit;
}

// Register custom page templates
add_filter('theme_page_templates', 'add_alpine_page_template');
function add_alpine_page_template($templates) {
    $templates['alpine-template.php'] = 'Alpine.js Application';
    return $templates;
}

// Load template from plugin
add_filter('template_include', 'load_alpine_page_template');
function load_alpine_page_template($template) {
    if(get_page_template_slug() === 'alpine-template.php') {
        $template = plugin_dir_path(__FILE__) . 'templates/alpine-template.php';
    }
    return $template;
}

// Enqueue built assets
add_action('wp_enqueue_scripts', 'enqueue_alpine_assets');
function enqueue_alpine_assets() {
    if (is_page() && get_page_template_slug() === 'alpine-template.php') {
        $asset_file = include plugin_dir_path(__FILE__) . 'build/index.asset.php';
        
        // Enqueue the built JS with dependencies
        wp_enqueue_script(
            'alpine-app',
            plugin_dir_url(__FILE__) . 'build/index.js',
            $asset_file['dependencies'],
            $asset_file['version'],
            true
        );
        
        // Enqueue the built CSS
        wp_enqueue_style(
            'alpine-styles',
            plugin_dir_url(__FILE__) . 'build/style.css',
            array(),
            $asset_file['version']
        );
    }
}

// Create page on activation
register_activation_hook(__FILE__, 'create_alpine_page');
function create_alpine_page() {
    $page_check = get_page_by_path('alpine-app');
    if (!$page_check) {
        $page = array(
            'post_title'    => 'Alpine.js Application',
            'post_name'     => 'alpine-app',
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

### Enqueuing Built Assets

The `@wordpress/scripts` package generates an asset file (`build/index.asset.php`) that contains:
1. An array of dependencies
2. A version number based on file content hash

Using this file for enqueuing ensures proper dependency management and automatic cache busting:

```php
function enqueue_alpine_assets() {
    // Load the asset file to get dependencies and version
    $asset_file = include plugin_dir_path(__FILE__) . 'build/index.asset.php';
    
    // Enqueue the built JavaScript
    wp_enqueue_script(
        'alpine-app',
        plugin_dir_url(__FILE__) . 'build/index.js',
        $asset_file['dependencies'],
        $asset_file['version'],
        true
    );
    
    // Enqueue the built CSS
    wp_enqueue_style(
        'alpine-styles',
        plugin_dir_url(__FILE__) . 'build/style.css',
        array(),
        $asset_file['version']
    );
}
```

### Cache Busting

The `@wordpress/scripts` package handles cache busting automatically in several ways:

1. **Content-based versioning**: The version number in `index.asset.php` is generated based on the content hash, changing whenever the file changes
2. **Long-term caching**: Production builds include content hashes in filenames (e.g., `index.123abc.js`)
3. **Dependency tracking**: Ensures proper loading order and updates dependencies when needed

This approach ensures:
- Browsers always load the latest version after updates
- Maximum caching for unchanged files
- No manual version management needed

## Advanced Techniques

### Multiple Editable Regions

To create multiple editable regions:

1. **Create separate post meta fields** for different content sections
2. Use custom Gutenberg blocks to edit these sections
3. In your template, retrieve and display each section:

```php
// In your template file
$header_content = get_post_meta(get_the_ID(), 'header_region', true);
$sidebar_content = get_post_meta(get_the_ID(), 'sidebar_region', true);

// Add to your Alpine.js data
$page_data['regions'] = array(
    'header' => $header_content,
    'sidebar' => $sidebar_content
);
```

Then in your template:

```html
<div class="header-region" x-html="page.regions.header"></div>
<div class="sidebar-region" x-html="page.regions.sidebar"></div>
```

### Custom Alpine Directives

You can register custom Alpine.js directives for common functionality:

```html
<script>
    document.addEventListener('alpine:init', () => {
        // Custom directive for WordPress formatting
        Alpine.directive('wpformat', (el, { expression }, { evaluate }) => {
            const content = evaluate(expression);
            if (content) {
                // Apply WordPress formatting
                el.innerHTML = content.replace(/\n/g, '<br>');
            }
        });
    });
</script>

<!-- Usage -->
<div x-wpformat="page.content"></div>
```

### Alpine.js Stores

For global state management across components:

```html
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.store('userData', {
            isLoggedIn: <?php echo is_user_logged_in() ? 'true' : 'false'; ?>,
            user: <?php echo json_encode(wp_get_current_user()); ?>
        });
    });
</script>

<!-- Usage anywhere in your template -->
<div x-data>
    <template x-if="$store.userData.isLoggedIn">
        <p>Welcome, <span x-text="$store.userData.user.display_name"></span>!</p>
    </template>
    <template x-if="!$store.userData.isLoggedIn">
        <p>Please log in to access all features.</p>
    </template>
</div>
```

### Site-Wide Header and Footer Override

If you want to completely override the theme's header and footer across your site, you can create a plugin that takes control of these elements regardless of the active theme.

#### File Structure for Site-Wide Header and Footer Override

```
your-plugin/
├── your-plugin.php              # Main plugin file with override logic
├── templates/
│   ├── custom-header.php        # Custom header replacement
│   ├── custom-footer.php        # Custom footer replacement
│   ├── home-template.php        # Home page template
│   └── archive-template.php     # Archive page template
└── assets/
    └── css/
        └── custom-style.css     # Custom styles for header/footer
```

#### Plugin-Based Template Override with Template Hierarchy

This approach allows you to override headers and footers site-wide while providing custom templates for specific pages:

```php
/**
 * Template Override System
 * 
 * Provides a way to override headers, footers, and specific page templates
 * regardless of the active theme.
 */

// Override specific templates based on conditions
add_filter('template_include', 'override_templates', 9999);
function override_templates($template) {
    // For home page, use your custom template
    if (is_front_page()) {
        $custom_template = plugin_dir_path(__FILE__) . 'templates/home-template.php';
        if (file_exists($custom_template)) {
            return $custom_template;
        }
    }
    
    // For archive pages, use a custom archive template
    if (is_archive() || is_home()) {
        $custom_template = plugin_dir_path(__FILE__) . 'templates/archive-template.php';
        if (file_exists($custom_template)) {
            return $custom_template;
        }
    }
    
    // For all other pages, return the original template
    // The header/footer will still be overridden below
    return $template;
}

// Override header site-wide
add_action('get_header', 'override_header', 9999);
function override_header($name) {
    // Prevent infinite loop
    remove_action('get_header', 'override_header', 9999);
    
    // Load your custom header instead
    $path = plugin_dir_path(__FILE__) . 'templates/custom-header.php';
    if (file_exists($path)) {
        include $path;
        // Exit to prevent the theme's header from loading
        $GLOBALS['wp_did_header'] = true; 
        return;
    }
    
    // If we got here, reattach our filter for future calls
    add_action('get_header', 'override_header', 9999);
}

// Override footer site-wide
add_action('get_footer', 'override_footer', 9999);
function override_footer($name) {
    // Similar approach to the header
    remove_action('get_footer', 'override_footer', 9999);
    
    $path = plugin_dir_path(__FILE__) . 'templates/custom-footer.php';
    if (file_exists($path)) {
        include $path;
        return;
    }
    
    add_action('get_footer', 'override_footer', 9999);
}
```

#### Header and Footer Override Flowchart

```
┌───────────────────────┐
│ WordPress Loads Theme │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐         ┌───────────────────────┐
│ Theme Calls get_header │─────────► Your Override Action  │
└───────────────────────┘         │ (Priority 9999)       │
                                   └───────────┬───────────┘
                                               │
┌───────────────────────┐                      │
│ Your Custom Header    │◄─────────────────────┘
│ (templates/custom-    │
│  header.php)          │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────┐         ┌───────────────────────┐
│ Theme Calls get_footer │─────────► Your Override Action  │
└───────────────────────┘         │ (Priority 9999)       │
                                   └───────────┬───────────┘
                                               │
┌───────────────────────┐                      │
│ Your Custom Footer    │◄─────────────────────┘
│ (templates/custom-    │
│  footer.php)          │
└───────────────────────┘
```

With this approach, you need to create these template files in your plugin:

1. **`templates/custom-header.php`**: Your site-wide header replacement
2. **`templates/custom-footer.php`**: Your site-wide footer replacement
3. **`templates/home-template.php`**: Complete template for the home page
4. **`templates/archive-template.php`**: Template for archive/blog pages

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

### Deployment Strategy

When deploying to production:

1. Include all necessary files:
   - All PHP files (templates, main plugin file, etc.)
   - The `build/` directory with compiled assets
   - `package.json` and `README.md` for documentation

2. You can exclude development files:
   - `node_modules/` directory
   - `src/` directory (if you include built assets)
   - Development configuration files (`.eslintrc`, etc.)

3. Create a deployment ZIP:
   ```bash
   # Example script to create a deployment ZIP
   mkdir -p dist
   zip -r dist/your-plugin.zip your-plugin.php templates/ includes/ build/ package.json README.md
   ```

## References

- [WordPress Theme Handbook: Page Templates](https://developer.wordpress.org/themes/template-files-section/page-template-files/)
- [Alpine.js Documentation](https://alpinejs.dev/start-here)
- [WordPress Developer Handbook](https://developer.wordpress.org/)
- [WordPress Plugin Development](https://developer.wordpress.org/plugins/)
- [WordPress Template Conditionals](https://developer.wordpress.org/themes/basics/conditional-tags/)
- [WordPress Scripts Package](https://developer.wordpress.org/block-editor/reference-guides/packages/packages-scripts/)
- [Alpine.js and SSR Best Practices](https://serversideup.net/alpinejs-and-server-side-rendering-a-perfect-match/)
- [Asset Versioning & Cache Busting in WordPress](https://css-tricks.com/strategies-for-cache-busting-css/)
