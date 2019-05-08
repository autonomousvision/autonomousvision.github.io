# Autonomous Vision Blog

This is the blog of the Autonomous Vision Group at MPI-IS Tübingen and University of Tübingen.
You can visit our blog at <https://autonomousvision.github.io>.
Also check out our [website](https://avg.is.tuebingen.mpg.de/) to learn more about our research.

## Instructions for Authors

To write a new blog entry, first register yourself as an author in [authors.yml](https://github.com/autonomousvision/autonomousvision.github.io/blob/master/_data/authors.yml).
Here, you can also add your email address and links to your social media accounts etc.

You can then create a new blog post by adding a markdown or html file in the [_posts](https://github.com/autonomousvision/autonomousvision.github.io/tree/master/_posts) folder.
Please use the format `YYYY-MM-DD-YOUR_TITLE.{md,html}` for naming the file. You can then create a yaml header where you specify the author, the category of the post, tags, etc. For more information, take a look at existing posts and the [Minimal Mistakes documentation](https://mmistakes.github.io/minimal-mistakes/docs/posts/).

If you want to include images or other assets, create a subfolder in the [assets/posts](https://github.com/autonomousvision/autonomousvision.github.io/tree/master/assets/posts) folder with the same name as the filename of your blog post (without extension).
You can simply reference your assets in your post using `{{ site.url }}/assets/posts/YYYY-MM-DD-YOUR_TITLE/` followed by the filename of the corresponding asset.
Make sure that you don't forget to include the `{{ site.url }}`! While the post while be rendered correctly without the `{{ site.url }}`, the images in the newsfeed will break if you don't include it.

Please keep the following things in mind when writing your posts:
- Don't use the public github repository as `origin`. Instead, use our internal (private) repository as `origin`. If you want to publish your post, add the public github repository as an additional remote called `public` (see [below](#publish-your-post) for details).
- Keep in mind that all your commits to master will appear in the public git history of our public repository. To keep this history clean, it might make sense to edit your post in a separate (private) branch and then merge this branch into `master`.
 
## Offline editing
When you do offline editing, you probably want to build the website offline for a preview.
To this end, you first have to [install Ruby and Jekyll](https://jekyllrb.com/docs/installation/).
Then, you have to install the dependencies (called [Gems](https://rubygems.org/)) for the website:
```
bundle
```

Now, you are ready to build and serve the website using
```
bundle exec jekyll serve
```

This command will build the website and serve it at <http://localhost:4000>.
When you save changes, the website will be automatically rebuilt in the background.
Note, however, that changes to `_config.yaml` will not be tracked which means that you have to restart the jekyll server after configuration changes.

## Publish your post
When you want to publish your post, first make sure that you have commited all your changes to the `master` branch and updated our internal git repository.


Then add the [public github repository](https://github.com/autonomousvision/autonomousvision.github.io) as a remote:
```
git remote add public git@github.com:autonomousvision/autonomousvision.github.io.git
```

You can now publish your post using
```
git push public master
```

## References
You can find more information here:

* [Jekyll documentation](https://jekyllrb.com/)
* [Minimal Mistakes documentation](https://mmistakes.github.io/minimal-mistakes/)
