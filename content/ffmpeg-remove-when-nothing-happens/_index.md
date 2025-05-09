+++
title = "Remove parts of videos when nothing happens"
+++


# Slice instagram video     :project:

**\*Removing Silence from FFmpeg Output: Automating Slicing for Instagram Shorts**

In today’s digital landscape, video content has become a vital tool for engaging audiences, particularly on platforms like Instagram. As creators aim to captivate viewers, the art of video editing has taken center stage. However, the process of manually slicing videos to extract the most interesting parts can be time-consuming and tedious. With the increasing demand for concise and engaging shorts, automating this slicing process using powerful tools like FFmpeg emerges as a game-changer.


### The Challenge of Manual Slicing

For creators dedicated to producing vibrant and engaging Instagram content, the task of slicing through hours of footage to find compelling clips can be overwhelming. Every second counts; therefore, losing time navigating through silence or less engaging content significantly hampers creativity and productivity. Manual slicing often involves tedious back-and-forth movement along the timeline, trying to discern which sections might resonate most with viewers. This not only draws out the editing process but can also lead to frustration when trying to maintain a consistent and captivating narrative.


### The Power of FFmpeg

FFmpeg is a versatile command-line tool that allows users to process video and audio files seamlessly. One of its extraordinary features is the ability to remove silence from audio tracks automatically—a boon for anyone looking to slice videos efficiently. By harnessing the power of FFmpeg to analyze audio, it becomes possible to eliminate segments of silence, allowing content creators to focus solely on the engaging parts of their videos.


### Approach

1.  **Prepare Your Environment:**
    -   Install FFmpeg on your system if you haven't already.

2.  **Identify Silence:**
    -   Use the `silencedetect` filter to identify silent sections in your video.

3.  **Automate Slicing:**
    -   Use `mpdecimate` filter to extract only the frames with motion and discard static frames.
    -   Combine commands to cut video segments based on identified motion or non-silent sections.

4.  **Export & Finalize:**
    -   Output the processed video in a suitable format for Instagram.
    -   Ensure the final video meets Instagram’s size and duration requirements.

5.  **Review & Edit:**
    -   Watch the sliced video for coherence and engagement.
    -   Make any necessary adjustments before posting.

This method allows you to streamline the editing process and create engaging Instagram videos that capture your audience's attention effectively.


### Step 1, pre-process the video     :ATTACH:

I will use one of my drawing videos as an example.

For simplicity - I will convert it to 10FPS gif.

[Source video](file:///Users/renatgalimov/Artwork Agile Drive/Org/data/C1/509929-A55E-42D9-9F78-1B36A4D3F558/source-video.webm)

I want to start with the most simplified approach - a monochrome video. This way we can visualize the decision-making process of the mpdecimate filter.

    ffmpeg -i IMG_1207.MOV -vf "fps=10,format=gray,lutyuv='y=if(gt(val,110), 255,0)',scale=240:-1" -vcodec libvpx -an -y source-video-monochrome.webm

[Monochrome source video](file:///Users/renatgalimov/Artwork Agile Drive/Org/data/C1/509929-A55E-42D9-9F78-1B36A4D3F558/source-video-monochrome.webm)
{{ webm(src="source-video-monochrome.webm") }}


### Step 2, understand the mpdecimate filter

To show how does mpdecimate work I will put tis hi and lo values to extremes and look at how the output would look like.

I will modify the filter we had before to reduce the difference between the `8x8` blocks.

Shortly, in the monochrome example maximum difference between two pixels is 255. I want to decrese the complexity and make it equal to 2.

I will do it with lutyuv='y=if(gt(val,110), 1,0)' filter. This will reduce the dynamic range of the video.

    ffmpeg -i IMG_1207.MOV -vf "fps=10,format=gray,lutyuv='y=if(gt(val,110), 255,254)',scale=240:-1" -vcodec libvpx -an -y source-video-low-dynamic-range.webm


[Low dynamic range video](file:///Users/renatgalimov/Artwork Agile Drive/Org/data/C1/509929-A55E-42D9-9F78-1B36A4D3F558/source-video-low-dynamic-range.webm)

You probably see just a white screen. This is because the difference between the pixels is too small.
But this difference is perfectly visible for the machine.

    ffmpeg -y -i source-video-low-dynamic-range.webm \
           -vf "fps=1,scale=16:-1,mpdecimate=hi=63:lo=15:frac=0.33,setpts=N/FRAME_RATE/TB,fps=1" \
           -an -loglevel debug -f null - 2>&1 | grep mpdecimate
