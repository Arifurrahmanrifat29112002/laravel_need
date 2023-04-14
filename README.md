# laravel_need

### Summernote CSS - CDN Link
  <!-- Summernote CSS - CDN Link -->
    <link href="https://cdn.jsdelivr.net/npm/summernote@0.8.18/dist/summernote.min.css" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/summernote@0.8.18/dist/summernote-lite.min.css" rel="stylesheet">
    <!-- Summernote JS - CDN Link -->
    <script src="https://cdn.jsdelivr.net/npm/summernote@0.8.18/dist/summernote-lite.min.js"></script>
    <script>
        $(document).ready(function() {
            $("#summernote").summernote({
                placeholder: 'describe your post',
                height: 400,
            });
            $('.select_js').select2();
        });
    </script>
    
    ### text_editor_image_delete
    
    <?php
      $post_data = Post::onlyTrashed()->find($post);
      unlink(base_path('public/uploads/post_thumbnail/' . $post_data->post_thumbnail));
      $post_description = $post_data->post_description;
      libxml_use_internal_errors(true);
      $dom = new \DomDocument();
      $dom->loadHtml('<?xml encoding="utf-8" ?>' . $post_description, LIBXML_HTML_NOIMPLIED | LIBXML_HTML_NODEFDTD); 
      // must include this to avoid font problem
      $images = $dom->getElementsByTagName('img');
      if (count($images) > 0) {
          foreach ($images as  $img) {
              $src = $img->getAttribute('src');
              $filename = last(explode("/", $src));
              unlink(base_path('public/uploads/post_thumbnail/' . $filename));
              # if the img source is 'data-url'
              if (preg_match('/data:image/', $src)) {
                  unlink(base_path('public/uploads/post_thumbnail/' . $filename));
              }
          }
      }

### text_editor_image_update


        $post_description = $request->post_description;
        $old_post_description = $post->post_description;
        if ($old_post_description !== $post_description) {
            libxml_use_internal_errors(true);
            $dom = new \DomDocument();
            $dom->loadHtml('<?xml encoding="utf-8" ?>' . $post_description, LIBXML_HTML_NOIMPLIED | LIBXML_HTML_NODEFDTD);    // must include this to avoid      font problem
            $images = $dom->getElementsByTagName('img');
            if (count($images) > 0) {
                foreach ($images as  $img) {
                    $src = $img->getAttribute('src');
                    if (!preg_match('/data:image/', $src)) {
                        $filename = last(explode("/", $src));
                        $oldfile = "uploads/post_thumbnail/$filename";
                        $mimetype = last(explode(".", $src));
                        $newFilename =
                            Str::limit($slug, 5) . '_' . Auth::guard('admin')->id() . '_' . time() .
                            Str::random(8) . '_' . Carbon::now()->format('Y');
                        $filepath = "uploads/post_thumbnail/$newFilename.$mimetype";
                        copy($oldfile, $filepath);
                        unlink(base_path("public/" . $oldfile));
                        $new_src = asset($filepath);
                        $img->removeAttribute('src');
                        $img->setAttribute('src', $new_src);
                    }
                    # if the img source is 'data-url'
                    if (preg_match('/data:image/', $src)) {
                        # get the mimetype
                        preg_match('/data:image\/(?<mime>.*?)\;/', $src, $groups);
                        $mimetype = $groups['mime'];
                        # Generating a random filename
                        $filename =
                            Str::limit($slug, 5) . '_' . Auth::guard('admin')->id() . '_' . time() .
                            Str::random(8) . '_' . Carbon::now()->format('Y');
                        $filepath = "uploads/post_thumbnail/$filename.$mimetype";
                        $image = Image::make($src)
                            ->encode($mimetype, 100)
                            ->save(public_path($filepath), 80);
                        $new_src = asset($filepath);
                        $img->removeAttribute('src');
                        $img->setAttribute('src', $new_src);
                    }
                }
            }
            # modified entity ready to store in database
            $post_description = $dom->saveHTML();
            $post->update([
                "post_description" => null,
            ]);
            $post->update([
                "post_description" => $post_description,
            ]);
        }
        
        
        ### text_editor_image_upload  
        <?php

        $post_description = $request->post_description;
        libxml_use_internal_errors(true);
        $dom = new \DomDocument();
        $dom->loadHtml('<?xml encoding="utf-8" ?>' . $post_description, LIBXML_HTML_NOIMPLIED | LIBXML_HTML_NODEFDTD);    // must include this to avoid font problem
        $images = $dom->getElementsByTagName('img');
        if (count($images) > 0) {
            foreach ($images as  $img) {
                $src = $img->getAttribute('src');
                # if the img source is 'data-url'
                if (preg_match('/data:image/', $src)) {
                    # get the mimetype
                    preg_match('/data:image\/(?<mime>.*?)\;/', $src, $groups);
                    $mimetype = $groups['mime'];
                    # Generating a random filename
                    $filename =
                        Str::limit($slug, 5) . '_' . Auth::guard('admin')->id() . '_' . time() .
                        Str::random(8) . '_' . Carbon::now()->format('Y');
                    $filepath = "uploads/post_thumbnail/$filename.$mimetype";
                    $image = Image::make($src)
                        ->encode($mimetype, 100)
                        ->save(public_path($filepath), 80);
                    $new_src = asset($filepath);
                    $img->removeAttribute('src');
                    $img->setAttribute('src', $new_src);
                }
            }
        }
# modified entity ready to store in database
$post_description = $dom->saveHTML();
