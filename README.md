# bgr-hugo

Hugo source repository for **[brokengadgets.io](https://brokengadgets.io)**. Pushes to `main` automatically build and publish the static site to [`brokengadgets/brokengadgets.github.io`](https://github.com/brokengadgets/brokengadgets.github.io) via GitHub Actions.

## Local Development

```bash
git clone --recurse-submodules git@github.com:brokengadgets/bgr-hugo.git
cd bgr-hugo
hugo server -D
```

Then open `http://127.0.0.1:1313` in your browser.
