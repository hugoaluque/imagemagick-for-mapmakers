# ImageMagick for Mapmakers

Use ImageMagick to create, edit, compose or convert digital images. This is a growing list of my most used ImageMagick commands that extend and enhance map making.

## The Magick

### Basics

Resize  
```shell
# resize all png files by 25% and save as jpg
ls *.png | while read file; do convert -resize 25% ${file} ${file%.*}.jpg; done

# force resize
convert shift_primemeridian_281_04_21_185611.gif -resize 1920x1080\! shift_primemeridian_1920_1080.gif

# resize by width.  
file='Esquire-Magazine-1990-05_0018.jpg'
convert "${file}" -gravity Center -resize 1920x\> +repage "${file%.*}.png"

# pixelate
convert chicago.png -scale 10% -filter point -scale 1000% chicago_pixelated.png
```

Change extent  
```shell
convert "${file}" -background none -gravity center -extent 1920x "${file%.*}_1920.png"
```

Convert to A4 pdf  
```shell
convert `ls *.webp` -colorspace gray -background white -level 60%,100% -page a4 shakespeare_a4_dark.pdf
```

Make blank image  
```shell
convert -size 1920x1080 xc:transparent box.png
```

Crop image  
```shell
convert hyp.png -gravity center -crop 960x540+0+0 +repage hyp_crop.png

convert -gravity center largeprint_kdp-page077.png -crop 1940x600+0+350 +repage largeprint_kdp-page077_1940x600.png

file='ne_50m_land.png'
convert "${file}" -gravity center -geometry 1920x1080^ -crop 1920x1080+0+0 "${file%.*}_1920_1080.png"
```

Split image into parts  
```shell
convert hyp.png -crop 25%x100% +repage hyp_part.png
```

Append  
```shell
# append split images in reverse order and flip horizontally
convert $(ls -r hyp_part-*.png) -flop +append hyp_append.png

# append 2 images
file1='Nick Mag #0 - Premiere 1990_0021.jpg'
file2='Nick Mag #0 - Premiere 1990_0022.jpg'
convert "${file1}" "${file2}" -gravity Center +append -resize 960x +repage ~/archive-mag/archive/"${file1%.*}.png"
```

### Colors & Effects

Black & white  
```shell
convert -colorspace gray manhattan.png -threshold 50% manhattan_threshold.png
```

Reduce the number of colors  
```shell
onvert manhattan.png -colors 4 manhattan_color4.png
```

Adjust image levels  
```shell
convert HYP_HR_SR_OB_DR.png -level 50%,100% hyp.png
```

Modulate brightness, saturation, hue  
```shell
convert -modulate 150,120,100 block_us.png block_us_modulated.png
```

Add a sketch effect with a canny edge detection layer  
```shell
convert hyp.png \( +clone -modulate 200 -canny 0x1+5%+20% -negate \) -gravity center -compose multiply -composite hyp_canny.png
```

Fill color for all files in directory  
```shell
for png_file in *.png; do
  convert "$png_file" -fill "#5F9EA0" -colorize 100% ../"${png_file}"
done
```

### Composite

Composite cloud cover image over Natural Earth raster  
```shell
convert hyp.png tcdc.png -gravity center -compose over -composite hyp_tcdc.png
```

Composite and adjust levels of each image in one command  
```shell
convert -size 1920x1080 xc:none \( hyp.png -level 50%,100% \) -gravity center -compose over -composite \( tcdc.png -level 50%,100% \) -gravity center -compose over -composite hyp_tcdc.png
```

Composite satellite images  
```shell
# compose methods: Over, Pegtop_Light, Screen, Darken_Intensity, Hardlight
convert -size 1920x1080 xc:none \( chicago2.png -level 50%,100% \) -gravity center -compose over -composite \( chicago2_2.png -level 50%,100% \) -gravity center -compose Darken_Intensity -composite hyp_tcdc.png
```

Montage images  
```shell
montage -background none $(ls -1 ~/svgeo/svg/ortho/*coastline* | sort -t_ -k4,4n) -tile 3x -geometry +0+0 -resize 640x -bordercolor none -border 50x50 -gravity center miff:- | convert - -gravity center -geometry 1920x1080^ -crop 1920x1080+0+0 +repage ne_50m_coastline.png

montage -background none screen1.jpg screen2.jpg screen3.jpg -tile 3x -geometry +0+0 -resize 640x -gravity center miff:- | convert - -gravity center -white-threshold 90% -geometry 1920x1080^ -crop 1920x1080+0+0 +repage websites.png

montage /home/steve/qgis-expressions/screens/20240307001511.png  -gravity center -crop 384x500+0+0 -geometry +0+0 -tile 5x1 - | convert - maps4.png

# random
montage $(ls *.png | shuf -n 15) -gravity center -crop 384x384+0+0 -geometry +0+0 -tile 5x3 output.png
```

Make a gif from a folder of files  
```shell
convert -delay 60 $PWD/*.png $(basename $PWD).gif
```

### Distort

Change the prime meridian 180*  
```shell
convert hyp.png -roll +960 hyp_roll.png
```

Apply an arc projection  
```shell
convert -size 1920x1080 xc:none \( hyp.png -virtual-pixel none -resize 50% -distort Arc '45' \) -gravity center -compose over -composite hyp_arc_45.png

convert -size 1920x1080 xc:none \( hyp.png -virtual-pixel none -resize 50% -rotate 180 -distort Arc '45 180' \) -gravity center -compose over -composite hyp_arc_45_180.png

convert -size 1920x1080 xc:none \( chicago.tif -virtual-pixel none -rotate 180 -distort Arc '45 180' \) -gravity center -compose over -composite chicago_arc.png
```

Apply a polar projection  
```shell
convert -size 1920x1080 xc:none \( hyp.png -virtual-pixel none -resize 50% -distort Polar '0' \) -gravity center -compose over -composite hyp_polar_0.png
```

### Animate

Make gif from svgs in a directory  
```shell
rm -rf ~/tmp/*
ls *.svg | while read file; do convert ${file} ~/tmp/${file%.*}.png; done
convert -delay 60 ~/tmp/*.png $(basename $PWD).gif
```

Loop gif nicely  
```shell
convert shift_primemeridian_1920_1080.gif -coalesce -duplicate 1,-2-1 -quiet -layers OptimizePlus -loop 0 shift_primemeridian_1920_1080_loop.gif
```

Convert video to gif  
```shell
convert weather_03_17_170534.mp4 -coalesce -layers OptimizePlus weather_03_17_170534.gif

file='Mazola.ia.mp4'
ffmpeg -y -i "$file" -r 10 -vf "fps=10,crop=in_w-50:in_h-50,scale=960:-1,setpts=PTS-STARTPTS" ~/archive-mag/archive/"${file%.*}.gif"
```

Extract frame  
```shell
convert input.gif[$selected_frame] output_frame.png
```

### Misc

Override cache limit  
```shell
sudo vim /etc/ImageMagick-6/policy.xml
<policy domain="resource" name="disk" value="8GB"/>
```

TO DO:

adding text captions...

