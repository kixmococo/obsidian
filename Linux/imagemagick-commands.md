# ImageMagick — Ubuntu Reference

## Install
```bash
sudo apt update
sudo apt install imagemagick
```

## Verify install
```bash
convert -version
# or, on ImageMagick 7:
magick -version
```

## Check installed version / repo version
```bash
apt-cache policy imagemagick
```

## Common commands

### Convert format
```bash
convert input.png output.jpg
```

### Resize an image
```bash
convert input.jpg -resize 50% output.jpg
convert input.jpg -resize 800x600 output.jpg
```

### Batch convert a folder
```bash
mogrify -format jpg *.png
```

### Compress / adjust quality
```bash
convert input.jpg -quality 85 output.jpg
```

### Crop
```bash
convert input.jpg -crop 300x300+50+50 output.jpg
```

### Rotate
```bash
convert input.jpg -rotate 90 output.jpg
```

### Create a GIF from images
```bash
convert -delay 20 -loop 0 frame*.png animation.gif
```

### Get image info
```bash
identify input.jpg
identify -verbose input.jpg
```

### Strip metadata (useful for shipping assets)
```bash
convert input.jpg -strip output.jpg
```

## Notes
- `convert`/`mogrify` are ImageMagick 6 commands; ImageMagick 7 consolidates into a single `magick` command (`magick input.png output.jpg`).
- `mogrify` overwrites files in place by default — use `-path` to output elsewhere instead of clobbering originals.
