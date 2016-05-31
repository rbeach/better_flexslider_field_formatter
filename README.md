
# Better Flexslider Field Formatter

## Purpose

I was working on a client project that was using the flexslider field formatter to display both images and video. They wanted the videos (using oEmbed to add videos from youtube) to render in a player, but they were rendering as broken images. This is because, while the flexslider module supports the file field type, it tries to render everything as an image.

```
function flexslider_fields_field_formatter_view($entity_type, $entity, $field, $instance, $langcode, $items, $display) {

  $element = array();
  if (count($items) > 0) {

    foreach ($items as $key => $item) {
      if(isset($item['item'])) continue;
      $tmp = $item;
      $item = array();
      $item['item'] = $tmp;

      // Setup the variables for calling theme_image_style
      if (isset($item['item']['uri'])) {
        $item['slide']['path'] = $item['item']['uri'];
      }
      if (isset($display['settings']['image_style'])) {
        $item['slide']['style_name'] = $display['settings']['image_style'];
      }
      if (isset($item['item']['width'])) {
        $item['slide']['width'] = $item['item']['width'];
      }
      if (isset($item['item']['height'])) {
        $item['slide']['height'] = $item['item']['height'];
      }
      if (isset($item['item']['alt'])) {
        $item['slide']['alt'] = $item['item']['alt'];
      }
      if (isset($item['item']['title'])) {
        $item['slide']['title'] = $item['item']['title'];
      }

      // Render the slide item
      // If not style set, we have to call theme_image since theme_image_style
      // doesn't auto-fallback to full size image
      if (!empty($item['slide']['style_name'])) {
        // Generate the HTML for the slide
        $item['slide'] = theme('image_style', $item['slide']);
      }
      else {
        // Generate the HTML for the slide
        $item['slide'] = theme('image', $item['slide']);
      }

      // Check caption settings
      if ($display['settings']['caption']) {
        $item['caption'] = filter_xss($item['item']['title']);
      }

      $items[$key] = $item;
    }

    $element = array(
      '#theme' => 'flexslider',
      '#items' => $items,
      '#settings' => $display['settings'],
    );
  }

  return $element;
}
```

My custom module creates a custom field formatter that re-writes this functionality to better support video. It will render images the same way as the flexslider field formatter, but will check to see if it is a video and if it is, will render the entity using a specified view mode instead of just using theme_image_style