Kirby
======================

A docset for [Kirby](http://getkirby.com) compiled by [Sam Nabi](http://samnabi.com).

For now, this only contains the [Cheat Sheet](http://getkirby.com/docs/cheatsheet). The rest of the docs will come in a later version.

To generate the docset, loop through [all the pages in the cheatsheet](https://github.com/getkirby/getkirby.com/tree/master/content/1-docs/9-cheatsheet) and look for pages with a template `cheatsheet.item`. I ran this within a Kirby install, using a custom template to extract the information and export each page as an HTML document. The related SQL query to populate Dash's `searchIndex` is also generated by the script.

```php
<?php
	// Create files even if there are no directories made yet
	function file_force_contents($dir, $contents){
	    $parts = explode('/', $dir);
	    $file = array_pop($parts);
	    $dir = '';
	    foreach($parts as $part)
	        if(!is_dir($dir .= "$part/")) mkdir($dir);
	    file_put_contents("$dir/$file", $contents);
	}
?>
<pre>
<?php foreach (page('docs/cheatsheet')->children()->index()->filterBy('template','cheatsheet.item') as $row) { ?>
	<?php
		// Set the variables for searchIndex
		$name = $row->title();
		$type = $row->return()->html() != '' ? 'Method' : 'Field';
		$path = $row->uri().'.html';

		// Build a basic HTML page
		$html = '<html lang="en"><head><title>'.$row->title().'</title></head><body>';
		$html .= '<h1>'.$row->title().'</h1>';
		$html .= $row->excerpt()->kirbytext();
		$html .= $type === 'Method' ? $row->return()->kirbytext() : '';
		$html .= $row->text()->kirbytext();
		$html .= '</body></html>';

		// Create the HTML page
		file_force_contents($path,$html);
	?>
INSERT OR IGNORE INTO searchIndex(name, type, path) VALUES ('<?php echo $name ?>', '<?php echo $type ?>', '<?php echo $path ?>');
<?php } ?>
</pre>
```

You'll have a list of SQL commands that you can copy-paste and execute to add items to the `searchIndex` all at once.

Manually move the generated HTML pages to `Kirby.docset/Contents/Resources/Documents/`. You should be ready to go.

Refer to the [Dash Docset Generation Guide](https://kapeli.com/docsets) for more details.
