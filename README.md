function custom_meta_box() {
	add_meta_box(
		'custom_meta_box_id', // unique ID
		'Custom Meta Box Title', // title of the meta box
		'custom_meta_box_callback', // callback function to display the meta box
		'your_custom_post_type_name', // post type
		'side', // location of the meta box
		'high' // priority of the meta box
	);
}
add_action( 'add_meta_boxes', 'custom_meta_box' );

function custom_meta_box_callback( $post ) {
	wp_nonce_field( basename( __FILE__ ), 'custom_meta_box_nonce' ); // security check
	$value = get_post_meta( $post->ID, '_custom_meta_key', true );
	?>
	<label for="custom_meta_field"><?php _e( 'Custom Meta Field Label', 'textdomain' ); ?></label>
	<input type="text" name="custom_meta_field" id="custom_meta_field" value="<?php echo esc_attr( $value ); ?>">
	<?php
}

function save_custom_meta_box( $post_id ) {
	if ( ! isset( $_POST['custom_meta_box_nonce'] ) || ! wp_verify_nonce( $_POST['custom_meta_box_nonce'], basename( __FILE__ ) ) ) {
		return $post_id;
	}
	if ( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE ) {
		return $post_id;
	}
	$post_type = get_post_type_object( get_post_type( $post_id ) );
	if ( ! current_user_can( $post_type->cap->edit_post, $post_id ) ) {
		return $post_id;
	}
	$new_value = ( isset( $_POST['custom_meta_field'] ) ? sanitize_text_field( $_POST['custom_meta_field'] ) : '' );
	update_post_meta( $post_id, '_custom_meta_key', $new_value );
}
add_action( 'save_post', 'save_custom_meta_box' );
