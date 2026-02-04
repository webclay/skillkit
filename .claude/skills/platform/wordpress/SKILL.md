---
name: wordpress
description: WordPress plugin development with PHP. Trigger words - wordpress, wp plugin, custom post type, wordpress rest api, wordpress admin, php plugin
---

# WordPress Plugin Development

Build WordPress plugins with PHP, hooks, and WordPress APIs.

## When to Use This Skill

- Building WordPress plugin functionality
- Adding custom post types or taxonomies
- Creating admin settings pages
- Building REST API endpoints for WordPress

## Plugin Structure

```
my-plugin/
├── my-plugin.php          # Main file (required)
├── uninstall.php          # Cleanup on uninstall
├── includes/
│   ├── class-activator.php
│   └── class-deactivator.php
├── admin/
│   ├── class-admin.php
│   └── js/admin.js
└── public/
    └── class-public.php
```

## Main Plugin File

```php
<?php
/**
 * Plugin Name:       My Plugin
 * Description:       What it does
 * Version:           1.0.0
 * Requires at least: 6.0
 * Requires PHP:      8.0
 * Author:            Your Name
 */

if ( ! defined( 'WPINC' ) ) {
    die;
}

define( 'MY_PLUGIN_VERSION', '1.0.0' );
define( 'MY_PLUGIN_PATH', plugin_dir_path( __FILE__ ) );

// Activation
register_activation_hook( __FILE__, 'my_plugin_activate' );
function my_plugin_activate() {
    // Create tables, set defaults
    flush_rewrite_rules();
}

// Deactivation
register_deactivation_hook( __FILE__, 'my_plugin_deactivate' );
function my_plugin_deactivate() {
    flush_rewrite_rules();
}

// Initialize
add_action( 'plugins_loaded', 'my_plugin_init' );
function my_plugin_init() {
    // Plugin code here
}
```

## Hooks (Actions & Filters)

```php
// ACTION: Do something
add_action( 'init', 'my_register_post_type' );

// FILTER: Modify data
add_filter( 'the_content', 'my_add_content' );
function my_add_content( $content ) {
    return $content . '<p>Added content</p>';
}

// Custom hooks for extensibility
do_action( 'my_plugin_before_save', $data );
$data = apply_filters( 'my_plugin_process_data', $data );
```

## Database Operations

```php
global $wpdb;

// SELECT (always use prepare)
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE user_id = %d",
        $user_id
    ),
    ARRAY_A
);

// INSERT
$wpdb->insert(
    $wpdb->prefix . 'my_table',
    array( 'user_id' => $user_id, 'data' => $data ),
    array( '%d', '%s' )
);
$id = $wpdb->insert_id;

// UPDATE
$wpdb->update(
    $wpdb->prefix . 'my_table',
    array( 'data' => $new_data ),
    array( 'id' => $id ),
    array( '%s' ),
    array( '%d' )
);

// DELETE
$wpdb->delete( $wpdb->prefix . 'my_table', array( 'id' => $id ) );
```

## Custom Post Type

```php
add_action( 'init', 'register_booking_cpt' );

function register_booking_cpt() {
    register_post_type( 'booking', array(
        'labels' => array(
            'name' => 'Bookings',
            'singular_name' => 'Booking',
        ),
        'public' => true,
        'show_in_rest' => true,
        'menu_icon' => 'dashicons-calendar-alt',
        'supports' => array( 'title', 'editor', 'thumbnail' ),
    ) );
}
```

## REST API Endpoint

```php
add_action( 'rest_api_init', 'my_register_routes' );

function my_register_routes() {
    register_rest_route( 'my-plugin/v1', '/items', array(
        'methods' => 'GET',
        'callback' => 'my_get_items',
        'permission_callback' => function() {
            return current_user_can( 'read' );
        },
    ) );

    register_rest_route( 'my-plugin/v1', '/items', array(
        'methods' => 'POST',
        'callback' => 'my_create_item',
        'permission_callback' => function() {
            return current_user_can( 'edit_posts' );
        },
    ) );
}

function my_get_items( $request ) {
    global $wpdb;
    $items = $wpdb->get_results( "SELECT * FROM {$wpdb->prefix}my_table" );
    return new WP_REST_Response( $items, 200 );
}

function my_create_item( $request ) {
    $title = sanitize_text_field( $request->get_param( 'title' ) );
    // Insert and return response
    return new WP_REST_Response( array( 'id' => $id ), 201 );
}
```

## Admin Settings Page

```php
add_action( 'admin_menu', 'my_add_menu' );

function my_add_menu() {
    add_menu_page(
        'My Plugin Settings',
        'My Plugin',
        'manage_options',
        'my-plugin',
        'my_settings_page',
        'dashicons-admin-generic',
        80
    );
}

function my_settings_page() {
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        <form action="options.php" method="post">
            <?php
            settings_fields( 'my_plugin_settings' );
            do_settings_sections( 'my-plugin' );
            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

## Security

```php
// Nonces (CSRF protection)
wp_nonce_field( 'my_action', 'my_nonce' );
if ( ! wp_verify_nonce( $_POST['my_nonce'], 'my_action' ) ) {
    die( 'Security check failed' );
}

// Sanitize input
$text = sanitize_text_field( $_POST['field'] );
$email = sanitize_email( $_POST['email'] );
$int = absint( $_POST['id'] );

// Escape output
echo esc_html( $text );
echo esc_url( $url );
echo esc_attr( $attr );

// Capability check
if ( ! current_user_can( 'manage_options' ) ) {
    wp_die( 'Unauthorized' );
}
```

## AJAX Handler

```php
add_action( 'wp_ajax_my_action', 'my_ajax_handler' );
add_action( 'wp_ajax_nopriv_my_action', 'my_ajax_handler' ); // Non-logged-in

function my_ajax_handler() {
    check_ajax_referer( 'my_nonce', 'nonce' );

    $data = sanitize_text_field( $_POST['data'] );

    // Process...

    wp_send_json_success( array( 'message' => 'Done' ) );
}
```

## Enqueue Scripts

```php
add_action( 'admin_enqueue_scripts', 'my_admin_scripts' );

function my_admin_scripts( $hook ) {
    if ( strpos( $hook, 'my-plugin' ) === false ) return;

    wp_enqueue_script(
        'my-admin-js',
        MY_PLUGIN_URL . 'admin/js/admin.js',
        array( 'jquery' ),
        MY_PLUGIN_VERSION,
        true
    );

    wp_localize_script( 'my-admin-js', 'myPlugin', array(
        'ajaxUrl' => admin_url( 'admin-ajax.php' ),
        'nonce' => wp_create_nonce( 'my_nonce' ),
    ) );
}
```

## Tips

- Always use `$wpdb->prepare()` for SQL
- Sanitize all input, escape all output
- Use nonces for form submissions
- Check capabilities before actions
- Use hooks for extensibility

## How to Verify

### Quick Checks
- Plugin activates without errors
- Admin pages load correctly
- AJAX requests succeed
- Data saves to database

### Common Issues
- "Headers already sent": Remove whitespace before `<?php`
- "Permission denied": Check `current_user_can()`
- SQL errors: Use `$wpdb->last_error` to debug
