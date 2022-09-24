# hover-image-in-opencart
Hover Image in Opencart
<?xml version="1.0" encoding="utf-8"?>
<modification>
  <name>Module Hover Over Image</name>
  <version>1.0</version>
  <author>Rupak Nepali</author>
  <link>https://webocreation.com</link>
  <code>webocreation_module_hover_over_image</code>
  <file path="admin/view/template/catalog/product_form.twig">
    <operation>
      <search><![CDATA[ <div class="tab-pane" id="tab-image"> ]]></search>
      <add position="after"><![CDATA[
            {% if image_back_status %}
              <div class="table-responsive">
                {# HoverOver Image #}
                <table class="table table-striped table-bordered table-hover">
<thead>
<tr>
<td class="text-left">{{ entry_image_back }}</td>
</tr>
</thead>
<tbody>
<tr>
<td class="text-left"><a href="" id="thumb-image-back" data-toggle="image" class="img-thumbnail"><img src="{{ thumb_back }}" alt="" title="" data-placeholder="{{ placeholder }}" /></a><input type="hidden" name="image_back" value="{{ image_back }}" id="input-image-back" /></td>
</tr>
</tbody>
</table>
</div>
            {% endif %}
            ]]>
      </add>
    </operation>
  </file>

  <file path="admin/controller/catalog/product.php">
    <operation>
      <search><![CDATA[if (isset($this->request->post['product_image'])) {]]></search>
      <add position="before"><![CDATA[
                // Hoverover Image
    $data['image_back_status'] = $this->config->get('module_hoveroverimage_status');
    if ($this->config->get('module_hoveroverimage_status')) {
		if (isset($this->request->post['image_back'])) {
			$data['image_back'] = $this->request->post['image_back'];
		} elseif (isset($this->request->get['product_id'])) {
			$image_back= $this->model_catalog_product->getBackProductImage($this->request->get['product_id']);
			if (!empty($image_back['image_back'])){
				$data['image_back'] = $image_back['image_back'];
			}	
		} else {
			$data['image_back'] = array();
		}
		if (isset($this->request->post['image_back']) && is_file(DIR_IMAGE . $this->request->post['image_back'])) {
			$data['thumb_back'] = $this->model_tool_image->resize($this->request->post['image_back'], 100, 100);
		} elseif (!empty($image_back) && is_file(DIR_IMAGE . $image_back['image_back'])) {
			$data['thumb_back'] = $this->model_tool_image->resize($image_back['image_back'], 100, 100);
		} else {
			$data['thumb_back'] = $this->model_tool_image->resize('no_image.png', 100, 100);
		}
    }
            ]]>      </add>
    </operation>
  </file>

  <file path="admin/language/en-gb/catalog/product.php">
    <operation>
      <search><![CDATA[ $_['error_unique'] ]]></search>
      <add position="before"><![CDATA[ $_['entry_image_back'] = 'Upload Hoverover Image'; ]]></add>
    </operation>
  </file>

  <file path="admin/model/catalog/product.php">
    <operation>
      <search><![CDATA[if (isset($data['image'])) {]]></search>
      <add position="before"><![CDATA[ 
        //Hoverover Image
        if ($this->config->get('module_hoveroverimage_status')) {
        if (isset($data['image_back'])) {
          $this->db->query("DELETE FROM " . DB_PREFIX . "product_to_image_back WHERE product_id = '" . (int)$product_id . "'");
          $this->db->query("INSERT INTO " . DB_PREFIX . "product_to_image_back SET product_id = '" . (int)$product_id . "', image_back = '" . $this->db->escape($data['image_back']) . "'");
        }
        }
         ]]>      </add>
    </operation>
  </file>
  <file path="admin/model/catalog/product.php">
    <operation>
      <search><![CDATA[public function getProductRelated($product_id) {]]></search>
      <add position="before"><![CDATA[ 
          //Hoverover Image
          public function getBackProductImage($product_id) {
            $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "product_to_image_back WHERE product_id = '" . (int)$product_id . "'");
            return $query->row;
          }
        ]]>      </add>
    </operation>
  </file>

  <file path="catalog/model/catalog/product.php">
    <operation>
      <search><![CDATA[public function getProductRelated($product_id) {]]></search>
      <add position="before"><![CDATA[ 
          //Hoverover Image
          public function getBackProductImage($product_id) {
            $query = $this->db->query("SELECT * FROM " . DB_PREFIX . "product_to_image_back WHERE product_id = '" . (int)$product_id . "'");
            return $query->row;
          }
        ]]>      </add>
    </operation>
  </file>

  <file path="catalog/controller/product/category.php">
    <operation>
      <search><![CDATA[$data['products'][] = array(]]></search>
      <add position="before"><![CDATA[ 
        //Hoverover Image
        $image_back_final = "";
        $data['image_back_status'] = $this->config->get('module_hoveroverimage_status');
        if ($this->config->get('module_hoveroverimage_status')) {
            $image_back = $this->model_catalog_product->getBackProductImage($result['product_id']);
            if (isset($image_back['image_back'])) {
              $image_back_final = $this->model_tool_image->resize($image_back['image_back'], $this->config->get('theme_' . $this->config->get('config_theme') . '_image_product_width'), $this->config->get('theme_' . $this->config->get('config_theme') . '_image_product_height'));
            } else {
              $image_back_final = "";
            }
        }
      ]]>      </add>
    </operation>
  </file>

  <file path="catalog/controller/product/category.php">
    <operation>
      <search><![CDATA[$data['products'][] = array(]]></search>
      <add position="after"><![CDATA['thumb_back'       => $image_back_final, //Hoverover Image]]></add>
    </operation>
  </file>

  <file path="catalog/view/theme/default/template/product/category.twig">
    <operation>
      <search><![CDATA[<img src="{{ product.thumb }}" alt="{{ product.name }}" title="{{ product.name }}" class="img-responsive"/>]]></search>
      <add position="replace"><![CDATA[ 
        {% if product.thumb_back %}
            <img src="{{ product.thumb }}" alt="{{ product.name }}" title="{{ product.name }}" onmouseover="this.src='{{ product.thumb_back }}'" onmouseout="this.src='{{ product.thumb }}'" class="img-responsive"/>
        {% else %}
            <img src="{{ product.thumb }}" alt="{{ product.name }}" title="{{ product.name }}" class="img-responsive"/>
        {% endif %}
        ]]>      </add>
    </operation>
  </file>

</modification>
