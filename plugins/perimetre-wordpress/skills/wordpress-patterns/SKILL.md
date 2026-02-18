---
name: wordpress-patterns
description: >
  WordPress development best practices for Périmètre projects. Use when building themes,
  custom post types, Gutenberg blocks, REST API extensions, or integrating headless
  WordPress with Next.js. Covers PHP patterns, security checklist, block development
  with @wordpress/scripts, and WP + Next.js integration.
---

# WordPress Development Patterns

Best practices for building WordPress sites the Périmètre way — secure, maintainable, and standards-compliant.

## Theme Structure

### Recommended Directory Layout

```
theme/
├── functions.php          # Bootstrapper only — require() other files
├── style.css              # Theme header comment
├── index.php              # Fallback template
├── inc/
│   ├── setup.php          # after_setup_theme, image sizes, menus
│   ├── enqueue.php        # wp_enqueue_scripts, wp_enqueue_block_assets
│   ├── post-types.php     # Custom post type registrations
│   ├── taxonomies.php     # Custom taxonomy registrations
│   ├── rest-api.php       # REST API extensions
│   └── blocks/            # Block registration files
├── blocks/
│   └── my-block/
│       ├── block.json     # Block metadata
│       ├── index.js       # Block editor script
│       ├── editor.scss    # Editor styles
│       └── style.scss     # Front-end styles
└── templates/             # Block templates (theme.json)
```

### functions.php — Bootstrapper Only

```php
<?php
// functions.php is a bootstrapper, not a dumping ground
require_once get_template_directory() . '/inc/setup.php';
require_once get_template_directory() . '/inc/enqueue.php';
require_once get_template_directory() . '/inc/post-types.php';
require_once get_template_directory() . '/inc/taxonomies.php';
require_once get_template_directory() . '/inc/rest-api.php';
```

---

## Custom Post Types

Always use `register_post_type()` inside the `init` hook. Use a unique prefix for `post_type` slugs.

```php
<?php
// inc/post-types.php
add_action( 'init', 'peri_register_post_types' );

function peri_register_post_types(): void {
    register_post_type( 'peri_project', [
        'labels'       => [
            'name'          => __( 'Projects', 'peri-theme' ),
            'singular_name' => __( 'Project', 'peri-theme' ),
        ],
        'public'       => true,
        'show_in_rest' => true,    // Required for Gutenberg support
        'supports'     => [ 'title', 'editor', 'thumbnail', 'excerpt' ],
        'rewrite'      => [ 'slug' => 'projects' ],
        'menu_icon'    => 'dashicons-portfolio',
        'has_archive'  => true,
    ] );
}
```

**Rules:**
- Always `show_in_rest => true` to enable REST API + Gutenberg support
- Always wrap labels in `__()` for i18n
- Use a short, consistent prefix (e.g., `peri_`) for all CPT slugs

---

## Gutenberg Block Development

Use `@wordpress/scripts` as the build tool. Define blocks with `block.json` metadata.

### block.json

```json
{
  "$schema": "https://schemas.wp.org/trunk/block.json",
  "apiVersion": 3,
  "name": "peri-theme/project-card",
  "version": "1.0.0",
  "title": "Project Card",
  "category": "theme",
  "description": "Displays a project summary card.",
  "supports": {
    "html": false,
    "align": [ "wide", "full" ]
  },
  "attributes": {
    "title": {
      "type": "string",
      "default": ""
    },
    "projectId": {
      "type": "number"
    }
  },
  "editorScript": "file:./index.js",
  "style": "file:./style.css",
  "editorStyle": "file:./editor.css"
}
```

### Block registration (PHP)

```php
<?php
// Register all blocks from the blocks directory
add_action( 'init', 'peri_register_blocks' );

function peri_register_blocks(): void {
    $blocks_dir = get_template_directory() . '/blocks';
    $block_dirs = glob( $blocks_dir . '/*', GLOB_ONLYDIR );

    foreach ( $block_dirs as $block_dir ) {
        register_block_type( $block_dir );
    }
}
```

### Block editor script (JS)

```js
// blocks/project-card/index.js
import { registerBlockType } from '@wordpress/blocks';
import { useBlockProps, InspectorControls } from '@wordpress/block-editor';
import { PanelBody, TextControl } from '@wordpress/components';
import metadata from './block.json';

registerBlockType( metadata.name, {
    edit( { attributes, setAttributes } ) {
        const blockProps = useBlockProps();
        return (
            <>
                <InspectorControls>
                    <PanelBody title="Settings">
                        <TextControl
                            label="Title"
                            value={ attributes.title }
                            onChange={ ( title ) => setAttributes( { title } ) }
                        />
                    </PanelBody>
                </InspectorControls>
                <div { ...blockProps }>
                    <p>{ attributes.title || 'Project Card' }</p>
                </div>
            </>
        );
    },
    save() {
        return null; // Dynamic block — render in PHP
    },
} );
```

### Dynamic block render (PHP)

```php
<?php
// inc/blocks/project-card.php
function peri_render_project_card( array $attributes, string $content ): string {
    $title = isset( $attributes['title'] ) ? esc_html( $attributes['title'] ) : '';

    ob_start();
    ?>
    <div class="wp-block-peri-theme-project-card">
        <h3><?php echo $title; ?></h3>
    </div>
    <?php
    return ob_get_clean();
}
```

---

## REST API Extensions

Use `register_rest_route()` inside the `rest_api_init` hook.

```php
<?php
// inc/rest-api.php
add_action( 'rest_api_init', 'peri_register_rest_routes' );

function peri_register_rest_routes(): void {
    register_rest_route( 'peri/v1', '/projects', [
        'methods'             => WP_REST_Server::READABLE,
        'callback'            => 'peri_get_projects',
        'permission_callback' => '__return_true',  // Public endpoint
        'args'                => [
            'per_page' => [
                'default'           => 10,
                'sanitize_callback' => 'absint',
                'validate_callback' => fn( $value ) => $value > 0 && $value <= 100,
            ],
        ],
    ] );

    register_rest_route( 'peri/v1', '/projects/(?P<id>\d+)', [
        'methods'             => WP_REST_Server::READABLE,
        'callback'            => 'peri_get_project',
        'permission_callback' => '__return_true',
        'args'                => [
            'id' => [
                'validate_callback' => fn( $id ) => is_numeric( $id ),
                'sanitize_callback' => 'absint',
            ],
        ],
    ] );
}

function peri_get_projects( WP_REST_Request $request ): WP_REST_Response {
    $per_page = $request->get_param( 'per_page' );

    $query = new WP_Query( [
        'post_type'      => 'peri_project',
        'posts_per_page' => $per_page,
        'post_status'    => 'publish',
        'no_found_rows'  => false,
    ] );

    $projects = array_map( 'peri_format_project', $query->posts );

    return rest_ensure_response( [
        'data'  => $projects,
        'total' => $query->found_posts,
    ] );
}
```

**Rules:**
- Always define `permission_callback` — never omit it
- Always sanitize and validate query args
- Return `WP_REST_Response` via `rest_ensure_response()`
- Use `WP_REST_Server::READABLE` / `CREATABLE` / `EDITABLE` / `DELETABLE` constants

---

## Headless WordPress + Next.js

When using WordPress as a headless CMS with a Next.js frontend:

### WordPress side

```php
<?php
// Allow cross-origin requests from Next.js dev + prod domains
add_filter( 'allowed_http_origins', 'peri_allowed_origins' );

function peri_allowed_origins( array $origins ): array {
    $origins[] = 'https://yoursite.com';
    $origins[] = 'http://localhost:3000';
    return $origins;
}

// Expose featured image URL in REST API
add_filter( 'rest_prepare_peri_project', 'peri_add_featured_image_url', 10, 3 );

function peri_add_featured_image_url(
    WP_REST_Response $response,
    WP_Post $post,
    WP_REST_Request $request
): WP_REST_Response {
    $featured_id = get_post_thumbnail_id( $post->ID );
    if ( $featured_id ) {
        $response->data['featured_image_url'] = wp_get_attachment_image_url(
            $featured_id,
            'large'
        );
    }
    return $response;
}
```

### Next.js side

```typescript
// lib/wordpress.ts
const WP_API = process.env.WORDPRESS_API_URL; // e.g. https://cms.yoursite.com/wp-json

export async function getProjects(perPage = 10) {
  const res = await fetch(
    `${WP_API}/peri/v1/projects?per_page=${perPage}`,
    { next: { revalidate: 3600 } } // ISR: revalidate every hour
  );

  if (!res.ok) throw new Error('Failed to fetch projects');
  return res.json();
}
```

---

## Hooks & Filters

### Always use priority and accepted args

```php
<?php
// ✅ Explicit priority and arg count
add_filter( 'the_content', 'peri_filter_content', 20, 1 );
add_action( 'save_post_peri_project', 'peri_on_project_save', 10, 3 );

// ❌ Avoid: implicit priority (hard to debug)
add_filter( 'the_content', 'peri_filter_content' );
```

### Remove hooks correctly

```php
<?php
// ✅ Remove with matching priority
remove_filter( 'the_content', 'wptexturize', 10 );

// ✅ Remove object method hook
remove_action( 'init', [ $instance, 'setup' ], 10 );
```

---

## Internationalization (i18n)

Use the correct function for each context:

```php
<?php
// Translatable string
__( 'Hello World', 'peri-theme' );

// Echo translated string
_e( 'Hello World', 'peri-theme' );

// Escaped + translated (for HTML output)
esc_html__( 'Hello World', 'peri-theme' );
esc_html_e( 'Hello World', 'peri-theme' );

// With context (disambiguate same string)
_x( 'Post', 'post type name', 'peri-theme' );

// Pluralization
_n( '%s item', '%s items', $count, 'peri-theme' );
```

**Rules:**
- Always use the theme text domain consistently
- Register it with `load_theme_textdomain()` in `after_setup_theme`
- Never concatenate translated strings — use `printf()` / `sprintf()`

---

## Performance

### WP_Query best practices

```php
<?php
// ✅ Only query what you need
$query = new WP_Query( [
    'post_type'              => 'peri_project',
    'posts_per_page'         => 10,
    'post_status'            => 'publish',
    'no_found_rows'          => true,  // Skip COUNT query when pagination not needed
    'update_post_meta_cache' => false, // Skip meta cache when not needed
    'update_post_term_cache' => false, // Skip term cache when not needed
    'fields'                 => 'ids', // Get only IDs when that's all you need
] );

// ❌ NEVER: Unbounded query
$query = new WP_Query( [
    'post_type'      => 'peri_project',
    'posts_per_page' => -1,  // Dangerous on large datasets
] );
```

### Avoid direct database queries

```php
<?php
// ❌ Avoid unless absolutely necessary
global $wpdb;
$results = $wpdb->get_results( "SELECT * FROM {$wpdb->posts}" );

// ✅ Use WP_Query or WP core APIs instead
$query = new WP_Query( [ 'post_type' => 'peri_project' ] );
```

If a direct query is unavoidable, always use prepared statements:

```php
<?php
global $wpdb;
// ✅ Prepared statement — prevents SQL injection
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT ID, post_title FROM {$wpdb->posts} WHERE post_type = %s AND post_status = %s LIMIT %d",
        'peri_project',
        'publish',
        10
    )
);
```

---

## Security Checklist

Run through this checklist before shipping any WordPress code:

- [ ] **Nonces** — All forms and AJAX requests use `wp_nonce_field()` / `check_admin_referer()` / `check_ajax_referer()`
- [ ] **Capability checks** — All admin actions verify `current_user_can()` before proceeding
- [ ] **Input sanitization** — All user input is sanitized (`sanitize_text_field()`, `absint()`, `sanitize_email()`, etc.)
- [ ] **Output escaping** — All output is escaped (`esc_html()`, `esc_attr()`, `esc_url()`, `wp_kses_post()`)
- [ ] **Prepared statements** — All direct DB queries use `$wpdb->prepare()`
- [ ] **File uploads** — Validated with `wp_check_filetype()`, stored outside webroot or with restricted access
- [ ] **REST API** — All non-public endpoints have proper `permission_callback`
- [ ] **No hardcoded credentials** — All secrets in `wp-config.php` or environment variables
- [ ] **HTTPS** — `is_ssl()` check or forced via server config
