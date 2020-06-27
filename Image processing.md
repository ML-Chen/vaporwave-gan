```sh
cd /d/Pictures/Reddit/VaporwaveArt

# Extract middle frame of video clips into a local video_clips folder
mkdir video_clips
for video_file in *.{mp4,mov,webm,mkv}; do
    filename="${video_file%.*}" # strip the file extension
    # -ss marks what second to get the frame from
    ffmpeg -ss "$(bc -l <<< "$(ffprobe -loglevel error -of csv=p=0 -show_entries format=duration "$video_file")*0.5")" -i "$video_file" -frames:v 1 "video_clips/${filename}.png"
done

find SafeVaporwaveArt -type f -exec sh -c 'new=$(echo "{}" | tr "/" "-" | tr " " "_"); mv "{}" "$new"' \;

# Extract files in subfolders into the main folder
for dir in ./*/; do
    # Trim leading ./ and trailing /
    dir_name="${dir:2:-1}"
    for file in "$dir_name"/*; do
        mv "$file" "${file/\//-}"
    done
    rmdir "$dir"
done

output_folder=/d/Pictures/vaporwave
for size in 256x256 512x512; do
    for output_format in png jpg; do
        for source_format in png jpg; do
            for dir in ./*/; do
                # Trim leading ./ and trailing /
                dir_name="${dir:2:-1}"
                output_dir=${output_folder}/${size}${output_format}/${dir_name}
                mkdir -p "$output_dir"
                magick mogrify -resize ${size}^ -gravity center -extent $size -path "$output_dir" -format $output_format ./"${dir_name}"/*.${source_format}
            done
            magick mogrify -resize ${size}^ -gravity center -extent $size -path ${output_folder}/${size}${output_format} -format $output_format ./*.${source_format}
        done
        for file in *.gif; do
            echo $file
            magick convert "$file"[0] -coalesce -resize $size^ -gravity center -extent $size -format $output_format "$output_folder/$size$output_format/${file::-4}.$output_format"
        done
    done
done
```