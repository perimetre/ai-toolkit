---
name: wordpress-patterns
description: >
  WordPress development best practices for Périmètre projects. Use when building themes,
  custom post types, Gutenberg blocks (native and ACF Pro), REST API extensions, or
  integrating headless WordPress with Next.js via WPGraphQL. Covers PHP patterns,
  security checklist, block development with @wordpress/scripts and ACF, WPGraphQL
  editorBlocks querying, WPML translation, and WP + Next.js integration.
---

# WordPress Development Patterns

Best practices for building WordPress sites the Périmètre way — secure, maintainable, and standards-compliant.

## Plugin Stack

### Traditional Sites
- **ACF Pro** — custom fields, repeaters, ACF block registration
- **WPML + ACF Multilingual** — translation management

### Headless Sites (WPGraphQL)
- **WPGraphQL** — core GraphQL API
- **WPGraphQL Content Blocks** (WP Engine) — adds typed `editorBlocks` to posts/pages
- **WPGraphQL for ACF** (v2.0+) — exposes ACF field groups as typed GraphQL types inside `editorBlocks`
- **WPML GraphQL** — adds `language`, `translations`, and language filtering at the post level

> Note: WPML operates at the post level, not the block level. Each translated post has its own
> translated block content — you don't filter blocks by language, you query a different post.

---

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

### ACF Blocks vs Native Blocks

Périmètre projects use **both** ACF blocks and native Gutenberg blocks. Choose the right type per block — never mix both approaches in a single block.

| | **ACF Block** | **Native Block** |
|---|---|---|
| Best for | Field-heavy blocks (repeaters, image fields, selects) | Content-oriented blocks with live inline editing |
| Editor UI | Sidebar-based (InspectorControls via ACF) | React `edit()` component with inline editing |
| Requires React | No | Yes |
| Build speed | Faster | Slower (requires JS build step) |
| GraphQL output | `editorBlocks` → ACF field group (e.g. `heroFields`) | `editorBlocks` → `attributes` object |

**Never mix ACF and native block patterns in the same block.**

Both produce clean, typed `editorBlocks` output in WPGraphQL — the frontend query pattern is identical.

### ACF Block Registration

```php
<?php
// inc/blocks/hero.php — register on acf/init, not init
add_action( 'acf/init', 'peri_register_acf_blocks' );

function peri_register_acf_blocks(): void {
    acf_register_block_type( [
        'name'            => 'hero',             // Prefixed automatically: acf/hero
        'title'           => __( 'Hero', 'peri-theme' ),
        'description'     => __( 'Full-width hero section.', 'peri-theme' ),
        'render_template' => get_template_directory() . '/blocks/hero/render.php',
        'category'        => 'theme',
        'icon'            => 'cover-image',
        'keywords'        => [ 'hero', 'banner' ],
        'supports'        => [ 'align' => false ],
    ] );
}
```

### ACF Block Render Template

```php
<?php
// blocks/hero/render.php
// Always use get_field(), never the_field() — the_field() outputs unescaped HTML
$title    = get_field( 'title' );
$subtitle = get_field( 'subtitle' );
$image    = get_field( 'image' );  // Returns array with 'url', 'alt', etc.
$cta_url  = get_field( 'cta_url' );
?>
<section class="wp-block-acf-hero">
    <?php if ( $title ) : ?>
        <h1><?php echo esc_html( $title ); ?></h1>
    <?php endif; ?>

    <?php if ( $subtitle ) : ?>
        <p><?php echo esc_html( $subtitle ); ?></p>
    <?php endif; ?>

    <?php if ( $image ) : ?>
        <img
            src="<?php echo esc_url( $image['url'] ); ?>"
            alt="<?php echo esc_attr( $image['alt'] ); ?>"
        />
    <?php endif; ?>

    <?php if ( $cta_url ) : ?>
        <a href="<?php echo esc_url( $cta_url ); ?>">
            <?php esc_html_e( 'Learn more', 'peri-theme' ); ?>
        </a>
    <?php endif; ?>
</section>
```

**ACF escaping rules:**
- Always `esc_html( get_field() )` for text output
- Always `esc_url( get_field() )` for URLs
- Always `esc_attr( get_field() )` for HTML attributes
- Never use `the_field()` — it echoes unescaped content

---

## WPGraphQL & Headless Architecture

Périmètre headless builds use WPGraphQL (+ companion plugins) as the primary API layer.

### Setup

Install and activate the full plugin stack (see Plugin Stack section above). In **WPGraphQL → Settings**, enable **Public Introspection** so `@graphql-codegen/cli` can read the schema. Disable Public Introspection in production — always run codegen against a staging environment.

### WordPress side — CORS

```php
<?php
// Allow cross-origin requests from Next.js dev + prod domains
add_filter( 'allowed_http_origins', 'peri_allowed_origins' );

function peri_allowed_origins( array $origins ): array {
    $origins[] = 'https://yoursite.com';
    $origins[] = 'http://localhost:3000';
    return $origins;
}
```

### `editorBlocks` Query Pattern

All blocks (both native and ACF) appear in `editorBlocks`. Use `__typename` to branch on block type.

```graphql
query GetPage($id: ID!) {
  page(id: $id, idType: URI) {
    title
    editorBlocks(flat: false) {
      __typename
      name
      renderedHtml
      innerBlocks {
        __typename
        name
        renderedHtml
      }

      # Native block — attributes object
      ... on CoreHeading {
        attributes {
          level
          content
        }
      }

      # Native block — paragraph
      ... on CoreParagraph {
        attributes {
          content
          dropCap
        }
      }

      # ACF block — field group exposed as typed object
      ... on AcfHero {
        heroFields {
          title
          subtitle
          ctaUrl
          image {
            node {
              sourceUrl
              altText
            }
          }
        }
      }

      # ACF block with repeater
      ... on AcfFeatureList {
        featureListFields {
          items {
            label
            description
            icon {
              node {
                sourceUrl
              }
            }
          }
        }
      }
    }
  }
}
```

**Key rules:**
- Images resolve to `MediaItem` nodes (not raw IDs) — always access via `node { sourceUrl altText }`
- ACF repeaters resolve as typed arrays — field names match the ACF field key (camelCased)
- Use `flat: false` to preserve `innerBlocks` nesting; use `flat: true` for flat lists

### WPML GraphQL

WPML operates at the post level. Query a specific language by passing the `language` arg, and access `translations` to get sibling posts.

```graphql
query GetPageFR($id: ID!) {
  page(id: $id, idType: URI, language: FR) {
    title
    language {
      code
      locale
    }
    translations {
      language {
        code
      }
      uri
    }
    editorBlocks(flat: false) {
      __typename
      name
      renderedHtml
    }
  }
}
```

### Next.js Fetch Pattern

```typescript
// lib/graphql.ts
const WP_GRAPHQL = process.env.WORDPRESS_GRAPHQL_URL; // e.g. https://cms.yoursite.com/graphql

export async function fetchGraphQL<T>(
  query: string,
  variables: Record<string, unknown> = {},
  revalidate = 3600
): Promise<T> {
  const res = await fetch(WP_GRAPHQL!, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query, variables }),
    next: { revalidate }, // ISR: revalidate every hour by default
  });

  if (!res.ok) throw new Error(`GraphQL fetch failed: ${res.status}`);

  const json = await res.json();
  if (json.errors) throw new Error(json.errors[0].message);
  return json.data as T;
}
```

### Codegen Configuration

```yaml
# codegen.yml — points at staging schema
schema:
  - https://cms-staging.yoursite.com/graphql
documents: 'src/**/*.graphql'
generates:
  src/lib/graphql/generated.ts:
    plugins:
      - typescript
      - typescript-operations
config:
  avoidOptionals: true
  nonOptionalTypename: true
```

Run `graphql-codegen` against staging — never production. Public Introspection must be enabled on the staging WP instance.

### REST API (Traditional Builds)

For non-headless builds, use `register_rest_route()` inside the `rest_api_init` hook:

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
}
```

**REST API rules:**
- Always define `permission_callback` — never omit it
- Always sanitize and validate query args
- Return `WP_REST_Response` via `rest_ensure_response()`
- Use `WP_REST_Server::READABLE` / `CREATABLE` / `EDITABLE` / `DELETABLE` constants

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
