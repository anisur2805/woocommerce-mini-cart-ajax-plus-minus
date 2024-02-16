# WooCommerce Mini Cart Plus Minus Button Click AJAX Update

![Alt desc](https://github.com/anisur2805/woocommerce-mini-cart-ajax-plus-minus/blob/main/test.gif)

1. Create a JS file, enqueue and localize it in you functions.php file.
```
function fab_business_shop_scripts() {
    wp_enqueue_script( 'ic-cart-ajax', get_template_directory_uri() . '/js/mini-cart-ajax-update.js', array('jquery'), '1.0', true );
    wp_localize_script( 'ic-cart-ajax', 'my_ajax_object', array(
            'ajax_url' => admin_url( 'admin-ajax.php' ),
            'nonce'    => wp_create_nonce( 'ic-mc-nc' ),
        ) 
    );
}
add_action( 'wp_enqueue_scripts', 'fab_business_shop_scripts' );


// Here is the AJAX call 
add_action( 'wp_ajax_ic_qty_update', 'ic_qty_update' );
add_action( 'wp_ajax_nopriv_ic_qty_update', 'ic_qty_update' );

function ic_qty_update() {
    $key    = sanitize_text_field( $_POST['key'] );
    $number = intval( sanitize_text_field( $_POST['number'] ) );

    $cart = [
        'count'      => 0,
        'total'      => 0,
        'item_price' => 0,
    ];

    if ( $key && $number > 0 && wp_verify_nonce( $_POST['security'], 'ic-mc-nc' ) ) {
        WC()->cart->set_quantity( $key, $number );
        $items              = WC()->cart->get_cart();
        $cart               = [];
        $cart['count']      = WC()->cart->cart_contents_count;
        $cart['total']      = WC()->cart->get_cart_total();
        $cart['item_price'] = wc_price( $items[$key]['line_total'] );
    }

    echo json_encode( $cart );
    wp_die();
}

// Here I modify two more hooks for plus/minus button
function ic_display_quantity_plus() {
    echo '<button type="button" class="ic-item-quantity-btn plus" data-type="plus"> +</button>';
}
add_action( 'woocommerce_after_quantity_input_field', 'ic_display_quantity_plus', 10, 2 );


add_action( 'woocommerce_before_quantity_input_field', 'ic_display_quantity_minus' );
function ic_display_quantity_minus() {
    echo '<button type="button" class="ic-item-quantity-btn minus" data-type="minus">-</button>';
}
```

2. In JS file insert this code
```
;(function ($) {
    $(document).ready(function () {
        $("body").on("click", ".ic-item-quantity-btn", function () {
            ic_quantity_update_buttons($(this))
        });

        $("body").on("blur", ".ic-cart-sidebar-wrapper_body input", function () {
            ic_quantity_update_input_blue($(this))
        })
	
	$('body').on('click', '.ic-cart-header-btn', function(e){
            e.preventDefault();
            $('body').addClass('active-mini-cart');
        });
        
        $('body').on('click', '.ic-cart-header-btn-close', function(e){
            $('body').removeClass('active-mini-cart');
        });

        var ic_quantity_update_send = true
        // Update cart on button click
        function ic_quantity_update_buttons(el) {
            if( ic_quantity_update_send ) {
            $(".ic-cart-sidebar-wrapper_body ul").addClass("loading")
            ic_quantity_update_send = false
            var wrap = $(el).closest(".woocommerce-mini-cart-item")
            var input = $(wrap).find(".qty")
            var key = $(wrap).data("key")
            var number = parseInt($(input).val())
            var type = $(el).data("type")
            if (type == "minus") {
                number--
            } else {
                number++
            }
            if (number < 1) {
                number = 1
            }

            $(input).val(number)
            var data = {
                action: "ic_qty_update",
                key: key,
                number: number,
                security: my_ajax_object.nonce
            }

            $.post(my_ajax_object.ajax_url, data, function (res) {
                var cart_res = JSON.parse(res)
                console.log( cart_res )
                $( ".ic-cart-sidebar-wrapper_body  p.woocommerce-mini-cart__total.total .amount" ).html(cart_res["total"]);
                $(wrap) .find(".ic-custom-render-total").html(cart_res["item_price"]);
                ic_quantity_update_send = true;
                $(".ic-cart-sidebar-wrapper_body ul").removeClass("loading")

                // IF YOU WANT TO GO WITH HEADER COUNT/PRICE UPDATE ENABLE BELOW LINE AND FIX YOUR SELECTOR
                // $('.ic-mid-nav-right ul li a .ic-cart-qty span.ic-qty').html( cart_res['count'] );
                // $('.ic-cart-header-btn .ic-total-price').html( cart_res['total'] );
            })
        }
        }

        // Update cart on input blur
        function ic_quantity_update_input_blue( input ) {
            $(".ic-cart-sidebar-wrapper_body ul").addClass("loading")
            ic_quantity_update_send = false
            var wrap = $( input ).closest(".woocommerce-mini-cart-item")
            var key = $(wrap).data("key")
            var number = parseInt($(input).val());
            if( !number || number <1){
                number = 1;
            } 

            $(input).val(number)
            var data = {
                action: "ic_qty_update",
                key: key,
                number: number,
                security: my_ajax_object.nonce
            }

            $.post(my_ajax_object.ajax_url, data, function (res) {
                var cart_res = JSON.parse(res)
                $( ".ic-cart-sidebar-wrapper_body  p.woocommerce-mini-cart__total.total .amount" ).html(cart_res["total"]);
                $( wrap ) .find( ".ic-custom-render-total" ) .html(cart_res["item_price"]);
                $( ".ic-cart-sidebar-wrapper_body ul" ).removeClass("loading")

                // IF YOU WANT TO GO WITH HEADER COUNT/PRICE UPDATE ENABLE BELOW LINE AND FIX YOUR SELECTOR
                // $('.ic-mid-nav-right ul li a .ic-cart-qty span.ic-qty').html( cart_res['count'] );
                // $('.ic-cart-header-btn .ic-total-price').html( cart_res['total'] );
            })
        }
    })
})(jQuery)
```
3. Here I modify *mini-cart.php* file based on my requirement
```
<?php
/**
 * Mini-cart
 *
 * Contains the markup for the mini-cart, used by the cart widget.
 *
 * This template can be overridden by copying it to yourtheme/woocommerce/cart/mini-cart.php.
 *
 * HOWEVER, on occasion WooCommerce will need to update template files and you
 * (the theme developer) will need to copy the new files to your theme to
 * maintain compatibility. We try to do this as little as possible, but it does
 * happen. When this occurs the version of the template file will be bumped and
 * the readme will list any important changes.
 *
 * @see     https://docs.woocommerce.com/document/template-structure/
 * @package WooCommerce\Templates
 * @version 5.2.0
 */

defined( 'ABSPATH' ) || exit;

do_action( 'woocommerce_before_mini_cart' ); ?>

<?php if ( ! WC()->cart->is_empty() ) : ?>

	<ul class="woocommerce-mini-cart cart_list product_list_widget <?php echo esc_attr( $args['list_class'] ); ?>">
		<?php
		do_action( 'woocommerce_before_mini_cart_contents' );

		foreach ( WC()->cart->get_cart() as $cart_item_key => $cart_item ) {

			$_product   = apply_filters( 'woocommerce_cart_item_product', $cart_item['data'], $cart_item, $cart_item_key );
			$product_id = apply_filters( 'woocommerce_cart_item_product_id', $cart_item['product_id'], $cart_item, $cart_item_key );

			if ( $_product && $_product->exists() && $cart_item['quantity'] > 0 && apply_filters( 'woocommerce_widget_cart_item_visible', true, $cart_item, $cart_item_key ) ) {
				$product_name      = apply_filters( 'woocommerce_cart_item_name', $_product->get_name(), $cart_item, $cart_item_key );
				$thumbnail         = apply_filters( 'woocommerce_cart_item_thumbnail', $_product->get_image(), $cart_item, $cart_item_key );
				$product_price     = apply_filters( 'woocommerce_cart_item_price', WC()->cart->get_product_price( $_product ), $cart_item, $cart_item_key );
				$product_permalink = apply_filters( 'woocommerce_cart_item_permalink', $_product->is_visible() ? $_product->get_permalink( $cart_item ) : '', $cart_item, $cart_item_key );

				?>
				<li data-ic_product_id="<?php echo esc_attr($product_id); ?>" data-key="<?php echo $cart_item_key; ?>" class="test woocommerce-mini-cart-item <?php echo esc_attr( apply_filters( 'woocommerce_mini_cart_item_class', 'mini_cart_item', $cart_item, $cart_item_key ) ); ?>">

					<?php if (empty($product_permalink)) : ?>
						<?php echo $thumbnail; ?>
					<?php else : ?>
						<a href="<?php echo esc_url( $product_permalink ); ?>">
							<?php echo $thumbnail; ?>
						</a>
					<?php endif; ?>

                    <div class="ic-mini-cart-count-price ic-mini-cart-title-input">
						<?php 
							echo wp_kses_post( $product_name );
						
							$product_price = apply_filters( 'woocommerce_cart_item_price', WC()->cart->get_product_price( $cart_item['data'] ), $cart_item, $cart_item_key );
							echo woocommerce_quantity_input(
								array(
									'input_value' => $cart_item['quantity'],
								),
								$cart_item['data'], false 
							) . $product_price;
						?>

						<div class="ic-custom-render-total">
							<?php echo get_woocommerce_currency_symbol() . $cart_item['line_total'].'.00'; ?>
						</div>
					</div>

					<?php echo wc_get_formatted_cart_item_data( $cart_item ); ?>
					
                    <?php
						echo apply_filters(
							'woocommerce_cart_item_remove_link',
							sprintf(
								'<a href="%s" class="remove remove_from_cart_button" aria-label="%s" data-product_id="%s" data-cart_item_key="%s" data-product_sku="%s">&times;</a>',
								esc_url( wc_get_cart_remove_url( $cart_item_key ) ),
								esc_attr__( 'Remove this item', 'woocommerce' ),
								esc_attr( $product_id ),
								esc_attr( $cart_item_key ),
								esc_attr( $_product->get_sku() )
							),
							$cart_item_key
						);
					?>
				</li>
				<?php
			}
		}

		do_action( 'woocommerce_mini_cart_contents' );
		?>
	</ul>

	<p class="woocommerce-mini-cart__total total">
		<?php
		/**
		 * Hook: woocommerce_widget_shopping_cart_total.
		 *
		 * @hooked woocommerce_widget_shopping_cart_subtotal - 10
		 */
		do_action( 'woocommerce_widget_shopping_cart_total' );
		?>
	</p>

	<?php do_action( 'woocommerce_widget_shopping_cart_before_buttons' ); ?>

	<p class="woocommerce-mini-cart__buttons buttons"><?php do_action( 'woocommerce_widget_shopping_cart_buttons' ); ?></p>

	<?php do_action( 'woocommerce_widget_shopping_cart_after_buttons' ); ?>

<?php else : ?>

	<p class="woocommerce-mini-cart__empty-message"><?php esc_html_e( 'No products in the cart.', 'woocommerce' ); ?></p>

<?php endif; ?>

<?php do_action( 'woocommerce_after_mini_cart' ); ?>
```
4. Here is the front-end part
```
<div class="ic-cart-header-btn">Show Cart</div>

    <div class="ic-cart-sidebar-wrapper">
        <div class="ic-cart-sidebar-wrapper_header">
            <p><?php _e( 'Your Cart', 'wordpress' ); ?></p>
            <div class="ic-cart-header-btn-close"><i class="ri-close-line"></i></div>
        </div>
        <div class="ic-cart-sidebar-wrapper_body">
            <div class="cartcontents">
                <div class="widget_shopping_cart_content">
                    <?php 
                        woocommerce_mini_cart( [ 'list_class' => 'my-css-class' ] );
                    ?>
                </div>
            </div>
        </div>
    </div>
```
5. Style
```

.ic-cart-sidebar-wrapper {
    position: fixed;
    right: 0;
    top: 0;
    width: 430px;
    background: #fff;
    z-index: 999999;
    height: 100%;
    transform: translateX(100%);
    transition: all 0.3s;
}
.active-mini-cart .ic-cart-sidebar-wrapper {
    transform: translateX(0);
}

.ic-cart-header-btn {
    background: #ddd;
    border-radius: 5px;
    display: inline-block;
    padding: 6px 12px;
    min-width: 100px;
    cursor: pointer;
    text-align: center;
}

.ic-cart-sidebar-wrapper .remove.remove_from_cart_button {
    position: absolute;
    right: 0;
}

.ic-cart-sidebar-wrapper .quantity + span.woocommerce-Price-amount.amount {
    position: absolute;
    top: 50px;
    left: 115px;
}

.ic-cart-sidebar-wrapper .quantity + span.woocommerce-Price-amount.amount::before {
    content: 'Price';
    margin-right: 5px;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_header {
    padding: 20px 30px;
    display: flex;
    align-items: center;
    justify-content: space-between;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_header .ic-cart-header-btn-close {
    background: #F3F5F6;
    border-radius: 8px;
    width: 40px;
    height: 40px;
    border: 0;
    outline: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body {
    padding: 20px 30px;
    border-top: 1px solid #EAEAEA;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul {
    max-height: 650px;
    overflow-x: hidden;
    margin-right: -15px;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item {
    display: flex;
    padding: 20px 0;
    position: relative;
    line-height: 28px;
    border-bottom: 1px solid #EAEAEA;
    margin-right: 10px;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item img {
    width: 90px;
    height: 90px;
    border-radius: 4px;
    margin-right: 20px;
}


.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item .quantity {
    margin-top: 8px;
    min-height: 35px;
    max-height: 35px;
    max-width: 90px;
    position: relative;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item .quantity {
    margin-top: 28px;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item .quantity button:first-child {
    left: 5px;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item .quantity button:last-child {
    right: 5px;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item .quantity button {
    width: 25px;
    height: 25px;
    line-height: 0;
    background: #EAEAEA;
    -webkit-border-radius: 50%;
    -moz-border-radius: 50%;
    border-radius: 50%;
    padding: 0;
    font-size: 18px;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body ul li.woocommerce-mini-cart-item.mini_cart_item .quantity .qty {
    height: 35px;
    padding: 0;
    width: 100%;
    border: 1px solid #EAEAEA;
    -webkit-border-radius: 60px;
    -moz-border-radius: 60px;
    border-radius: 60px;
    font-size: 15px;
    line-height: 22px;
    color: #171717;
    text-align: center;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body .woocommerce-mini-cart__total {
    line-height: 35px;
    font-weight: 500;
    color: #171717;
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
}

.ic-cart-sidebar-wrapper .ic-custom-render-total {
    position: absolute;
    right: 0;
    bottom: 20px;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body .woocommerce-mini-cart__buttons {
    position: absolute;
    bottom: 35px;
    left: 30px;
    right: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
}

.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body .woocommerce-mini-cart__buttons a {
    flex: 1;
    text-align: center;
    padding: 15px 35px;
    border: 2px solid #00627A;
    filter: drop-shadow(0px 5px 20px rgba(245, 195, 75, 0.15));
    -webkit-border-radius: 5px;
    -moz-border-radius: 5px;
    border-radius: 5px;
    font-weight: 500;
    font-size: 15px;
    line-height: 22px;
    background-color: transparent;
    color: #171717;
    display: block;
    -webkit-transition: all 0.3s ease;
    -moz-transition: all 0.3s ease;
    -o-transition: all 0.3s ease;
    transition: all 0.3s ease;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body .woocommerce-mini-cart__buttons a:last-child {
    box-shadow: 0px 5px 20px rgba(0, 98, 122, 0.15);
    background: #00627A;
    color: #fff;
}
.ic-cart-sidebar-wrapper .ic-cart-sidebar-wrapper_body .woocommerce-mini-cart__buttons a:hover {
    box-shadow: 0px 5px 20px rgba(0, 98, 122, 0.15);
    background: #00627A;
    color: #fff;
}
```
