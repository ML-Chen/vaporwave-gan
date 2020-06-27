# Image processing

Download media from [r/VaporwaveArt](http://reddit.com/r/VaporwaveArt) using [Reddit Media Downloader](https://github.com/shadowmoose/RedditDownloader). We configured it with the PushShift: Subreddit Submissions source. As of June 24, 2020, this involved downloading 117 GB and 21,395 files, and took about 12 hours on a 60 Mb/s connection.

Then, we want to process these files into images of uniform size and image format. Some files will be video files or GIFs. We extract one frame from the middle of the videos, and the first frame of the GIFs.

I used a Bash script which I ran in Windows Subsystem for Linux on Windows 10 v2004 and Git Bash. Make sure that Windows Subsystem for Linux has ImageMagick installed. You can install it in Ubuntu with `sudo apt install imagemagick`. As of Ubuntu 20.04, the latest version of ImageMagick available is ImageMagick 6. If you have ImageMagick 7, you may have to write `magick convert` and `magick mogrify` instead of `convert` and `mogrify`.

We will filter out all the images with Zalgo titles like u̧͉̱̠̤̘̓̾͋͋̋̓n̳̘͝c̉̀͋͂ͭ͑͡ỏn̹̲̠̘̙ͫ͌̀s͕̜̘̖̎͂͠c̫̗͚͊ͫ̄̔̾͠i̹̞͗̌̿̏͂͢o͉̯̟̮͑̑̈́ͣ̿͑ṵ͈̥͈͕ͩ͛s͖̘̭͚̽̈͋̾ ̨̫i͎͎̖̟̻̮̔n̪ͭ̎̇v̴͙̖̳͙ͫe͈̪̩̊̒̐̉̓͞s̱̫̘̣̈̇̅͠t̋̽̎ͧ͟ḯ̙͍̩̲̫ͣͧͭ͂̾ͭ - (Lennobowski).jpg
because if such a file is in a folder, Windows Subsystem for Linux might won't be able to recognize many of the other files in the folder, and complain about an "Input/output error".

```sh
# cd to the directory in which the r/VaporwaveArt images were downloaded
cd /d/Pictures/Reddit/VaporwaveArt
# In Git Bash, move all the files names that can be recognized to a folder called SafeVaporwaveArt
find . -exec mv {} ../SafeVaporwaveArt/ \;

# Extract middle frame of video clips into a local video_clips folder
for video_file in *.{mp4,mov,webm,mkv}; do
    filename="${video_file%.*}" # strip the file extension
    # -ss marks what second to get the frame from
    ffmpeg -ss "$(bc -l <<< "$(ffprobe -loglevel error -of csv=p=0 -show_entries format=duration "$video_file")*0.5")" -i "$video_file" -frames:v 1 "${filename}.png"
done

# Extract files in subfolders into the main folder
for dir in ./*/; do
    # Trim leading ./ and trailing /
    dir_name="${dir:2:-1}"
    for file in "$dir_name"/*; do
        mv "$file" "${file/\//-}"
    done
    rmdir "$dir"
done

# Crop the images to a square, as a copy saved in $output_folder
output_folder=/mnt/d/Pictures/vaporwave
mkdir "$output_folder"
for size in 256x256 512x512; do
    for output_format in png jpg; do
        for source_format in png jpg; do
            mogrify -resize ${size}^ -gravity center -extent $size -path ${output_folder}/${size}${output_format} -format $output_format ./*.${source_format}
        done
        # Crop the first frame of each GIF as a copy
        for file in *.gif; do
            convert "$file"[0] -coalesce -resize $size^ -gravity center -extent $size -format $output_format "$output_folder/$size$output_format/${file::-4}.$output_format"
        done
    done
done
```

The ffmpeg script is based on https://stackoverflow.com/a/35026487/5139284, CC-BY-SA 4.0.
