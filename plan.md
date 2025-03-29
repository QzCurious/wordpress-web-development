# WordPress Plugin Development Plan: Alpine.js & TailwindCSS

## Project Overview

We'll develop a WordPress plugin that allows customization of a home page template with Alpine.js and TailwindCSS. The plugin will include:

- Custom home page template
- Custom header template that retrieves WordPress menu data
- Custom block to retrieve ACF fields
- Testimonial section using custom post type
- Editable hero section using Gutenberg editor
- Blog post section with category tabs

## Plugin Structure

```
alpine-tailwind-templates/
├── build/                       # Built assets (generated via build process)
│   ├── index.js                 # Main JS bundle with Alpine
│   ├── blocks.js                # Gutenberg blocks
│   └── style.css                # Compiled TailwindCSS
├── src/
│   ├── js/
│   │   ├── index.js             # Main JS entry point
│   │   └── components/          # Alpine components
│   ├── blocks/                  # Custom blocks
│   │   ├── index.js             # Block entry
│   │   └── acf-block/           # ACF data block 
│   └── css/
│       └── style.css            # TailwindCSS entry
├── templates/
│   ├── home-template.php        # Custom home page template
│   ├── header-template.php      # Custom header
│   └── partials/                # Template parts
├── includes/
│   ├── acf-fields.php           # ACF field registration
│   ├── post-types.php           # Custom post type registration
│   └── template-functions.php   # Helper functions
├── alpine-tailwind-templates.php # Main plugin file
├── package.json                 # Dependencies and scripts
└── tailwind.config.js           # TailwindCSS config
```

## Build Tools & Development Workflow

### @wordpress/scripts

We'll use `@wordpress/scripts` as our primary build tool, which provides:

- Webpack configuration out of the box
- Babel for JavaScript transpilation
- PostCSS processing for CSS
- ESLint and Stylelint for code quality
- Built-in support for WordPress development

This eliminates the need to write custom webpack configurations while providing optimized build processes for WordPress development.

### Package.json Configuration

```json
{
  "name": "alpine-tailwind-templates",
  "version": "1.0.0",
  "description": "WordPress plugin with Alpine.js and TailwindCSS templates",
  "scripts": {
    "build": "wp-scripts build src/js/index.js src/blocks/index.js --output-path=build",
    "start": "wp-scripts start src/js/index.js src/blocks/index.js --output-path=build",
    "lint:css": "wp-scripts lint-style",
    "lint:js": "wp-scripts lint-js",
    "format": "wp-scripts format"
  },
  "dependencies": {
    "alpinejs": "^3.12.0"
  },
  "devDependencies": {
    "@wordpress/scripts": "^26.0.0",
    "autoprefixer": "^10.4.14",
    "postcss": "^8.4.23",
    "tailwindcss": "^3.3.2"
  }
}
```

### Development Workflow

1. **Setup and Installation**:
   ```bash
   # Clone repository or create plugin directory
   mkdir alpine-tailwind-templates
   cd alpine-tailwind-templates
   
   # Initialize npm
   npm init -y
   
   # Install dependencies
   npm install --save alpinejs
   npm install --save-dev @wordpress/scripts tailwindcss postcss autoprefixer
   
   # Initialize TailwindCSS
   npx tailwindcss init
   ```

2. **Development Process**:
   ```bash
   # Start development server with hot reloading
   npm run start
   
   # WordPress development server
   cd /path/to/wordpress
   wp server
   ```

3. **Daily Development Cycle**:
   - Edit source files in the `src` directory
   - @wordpress/scripts watches for changes and rebuilds automatically
   - Browser refreshes to show changes (with BrowserSync or similar)
   - Commit changes to version control regularly

4. **Build for Production**:
   ```bash
   # Create optimized production build
   npm run build
   ```

### Asset Processing Pipeline

1. **JavaScript Processing**:
   - ES6+ features transpiled with Babel
   - Modules bundled with Webpack
   - Dependencies resolved automatically
   - Source maps generated for debugging

2. **CSS Processing**:
   - TailwindCSS compiled and optimized
   - PostCSS processes and adds vendor prefixes
   - CSS minified for production

3. **Asset Optimization**:
   - Production builds minified and optimized
   - Cache busting with content hashes
   - Dead code elimination
   - Automatic dependency management

## Development Phases

### Phase 1: Plugin Setup and Build System

1. Initialize plugin structure
2. Configure NPM and dependency installation:
   - Alpine.js
   - TailwindCSS
   - @wordpress/scripts
3. Configure TailwindCSS:
   - Create basic configuration
   - Set up PostCSS integration
   - Configure content sources for purging
4. Setup build process for asset compilation
5. Create main plugin file with hooks

### Phase 2: WordPress Global Styles Integration

1. Create PHP helper function to extract WordPress color palette:
   ```php
   // Add to includes/template-functions.php
   function get_wp_colors_for_tailwind() {
       if (function_exists('wp_get_global_settings')) {
           $global_settings = wp_get_global_settings();
           $palette = $global_settings['color']['palette']['theme'] ?? [];
           
           $colors = [];
           foreach ($palette as $color) {
               if (isset($color['slug']) && isset($color['color'])) {
                   $colors[$color['slug']] = $color['color'];
               }
           }
           
           return $colors;
       }
       
       // Fallback for older WordPress
       return [
           'primary' => '#3490dc',
           'secondary' => '#38c172',
           'accent' => '#9561e2',
       ];
   }
   ```

2. Add CSS variables injection for TailwindCSS:
   ```php
   // Generate CSS variables for direct use in templates
   function inject_wp_colors_as_css_vars() {
       $colors = get_wp_colors_for_tailwind();
       
       echo '<style id="wp-tailwind-colors">';
       echo ':root {';
       
       foreach ($colors as $name => $value) {
           echo "--wp-" . esc_attr($name) . ': ' . esc_attr($value) . ';';
       }
       
       echo '}';
       echo '</style>';
   }
   add_action('wp_head', 'inject_wp_colors_as_css_vars', 5);
   ```

3. Configure TailwindCSS to use WordPress colors:
   ```js
   // tailwind.config.js
   module.exports = {
     content: [
       './templates/**/*.php',
       './src/**/*.js',
     ],
     theme: {
       extend: {
         colors: {
           // Core colors using CSS variables with fallbacks
           'primary': 'var(--wp-primary, #3490dc)',
           'secondary': 'var(--wp-secondary, #38c172)',
           'accent': 'var(--wp-accent, #9561e2)',
           'background': 'var(--wp-background, #ffffff)',
           'text': 'var(--wp-text, #333333)',
         },
       },
     },
     plugins: [],
   }
   ```

4. Add helper function for theme developers (optional):
   ```php
   function wp_theme_color($color_name, $fallback = '') {
       $fallback = !empty($fallback) ? $fallback : '';
       return "var(--wp-" . esc_attr($color_name) . ', ' . esc_attr($fallback) . ')';
   }
   ```

### Phase 3: ACF Setup

1. Create testimonial custom post type
2. Define ACF fields for testimonials:
   - Name
   - Position
   - Content
   - Image
3. Create home-page-blog-category post type with fields:
   - Category selection
   - Order
   - Tab title

### Phase 4: Template Development

1. Create header template with Alpine.js integration:
   - Retrieve WordPress menu data via wp_get_nav_menu_items()
   - Implement responsive menu with Alpine.js
2. Create home page template structure:
   - Hero section container (Gutenberg editable)
   - Testimonials section
   - Blog posts with category tabs

### Phase 5: Custom Block Development

1. Create ACF block to display field data
2. Configure block for editor and frontend rendering
3. Ensure blocks properly enqueue required assets

### Phase 6: Home Page Template Sections

1. Hero Section:
   - Create container for Gutenberg content
   - Set default blocks (heading, paragraph, button)
   - Make fully editable by WordPress users

2. Testimonials Section:
   - Query testimonial post type
   - Implement slider/carousel with Alpine.js
   - Stylize with TailwindCSS

3. Blog Category Tabs:
   - Retrieve blog category settings from ACF
   - Implement tab switching with Alpine.js
   - Query posts by selected categories
   - Display up to 3 posts per category

### Phase 7: Asset Integration

1. Properly enqueue built assets
2. Configure conditional loading (only on template pages)
3. Implement cache busting

## Technical Specifications

### Alpine.js Implementation

- Use Alpine.js for interactive elements:
  - Mobile menu toggle
  - Testimonial slider/carousel
  - Blog category tabs
- Initialize Alpine data stores for global state
- Structure components for proper data scoping

### TailwindCSS Integration

- Configure TailwindCSS for WordPress compatibility
- Leverage WordPress Global Styles through CSS variables
- Create utility classes for common elements
- Ensure proper purging for production

### WordPress Integration

- Register templates using `theme_page_templates` filter
- Load templates with `template_include` filter
- Properly enqueue scripts and styles
- Use wp_localize_script for passing data to Alpine.js

### ACF Integration

- Register fields programmatically
- Create field groups for different sections
- Implement proper data retrieval pattern
- Handle fallbacks for empty fields

## Example Component: Blog Tabs

```javascript
// src/js/components/blog-tabs.js
export default function blogTabs() {
  return {
    tabs: [],
    activeTab: null,
    
    init() {
      this.tabs = window.alpineTemplateData?.blogTabs || [];
      this.activeTab = this.tabs.length > 0 ? this.tabs[0].id : null;
    }
  }
}
```

## Deployment Checklist

1. Run production build (`npm run build`)
2. Verify all assets are properly minified
3. Check TailwindCSS purging for optimal CSS size
4. Package plugin with all required files
5. Test on clean WordPress installation
6. Document usage instructions