I decided a while ago that I wanted to to start a blog, but despite GitHub Pages natively supporting Jekyll, I was put off because I didn't want to setup Ruby and the other dependencies just for a static site. Sometime later I was reading [this article](https://tenthousandmeters.com/blog/python-behind-the-scenes-12-how-asyncawait-works-in-python) and I noticed it said:

> Proudly powered by Pelican, which takes great advantage of Python.

Python is my main language so, Pelican was perfect.

I started playing with it in a [toy repository](https://github.com/skarfie123/ghp-pelican). I liked it a lot, but the one thing I didn't like was that the publish step used another tool, [`ghp-import`](https://github.com/c-w/ghp-import), to push the rendered site to the `gh-pages` branch, and it had to be run **manually**. I decided to experiment with GitHub Actions, which recently started allowing Pages deployments via Actions. In another [toy repository](https://github.com/skarfie123/gh-actions) I tested the basics:

- python and pip
- pip can install packages
- can run a make command

That all, worked fine, so I started experimenting with building a pelican site and publishing to Pages. You can find the workflow [here](https://github.com/skarfie123/skarfie123.github.io/blob/main/.github/workflows/deploy.yml). It is heavily based on the [Jekyll example](https://github.com/actions/starter-workflows/blob/main/pages/jekyll.yml) in structure, and the use of the first party actions:

- [actions/checkout](https://github.com/marketplace/actions/checkout)
- [actions/configure-pages](https://github.com/marketplace/actions/configure-github-pages)
- [actions/upload-pages-artifact](https://github.com/marketplace/actions/upload-github-pages-artifact)
- [actions/deploy-pages](https://github.com/marketplace/actions/deploy-github-pages-site)

The difference is in the pelican build part. I wanted to avoid using the 3rd party pelican build actions from the marketplace, because I saw some that required generating and feeding in a GitHub token, and most were based on the old branch based Pages method. I felt there should be a simpler way, and It turned out I was right! In the end [this](https://github.com/skarfie123/skarfie123.github.io/blob/18747ba4ae8b816f274dcb8679c1136ede5c9847/.github/workflows/deploy.yml#L28-L30) was all it needed:

```yml
- run: pip install "pelican[markdown]"
- name: Generate Pelican
  run: pelican content -o output -s publishconf.py -e SITEURL='"${{ steps.pages.outputs.base_url }}"'
```

In the end I stopped using the default Makefile. It was easier to pass in the `SITEURL` in the build command, and since I mainly use Windows, I didn't use it much locally.

This site is now published automatically whenever I push a commit to `main`! ✅
