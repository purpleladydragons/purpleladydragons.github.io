- not fourier transform
- "fingerprints" of audio sample and database tracks
- they call them hash results, so they must be consistent?
- but it's not even immediately clear what to use so they had to decide that
- they had some attributes they thought we be good metrics for how good a fingerprint metric would be 
-   temporally localized, translation invariant, robust, and sufficiently entropic
-      hell does that mean? temporal locality means that distant events should not affect hash. i guess you take a hash of a small, local bit
-      translation invariant means that matching content (i assume this means music/lyrics etc) should not care about position in song/track. i think this basically means if you have a repeating chorus or set of chords, it doesn't matter where in the mp3 it occurs
-      robust means that fingerprints from clean database track should be reproducible even in the dirty sample tracks (ngl i would've thought opposite that you noise up the clean track)
-      entropy means that the fingerprints are unique enough and informative enough that we don't get too many false positives
- okay so how do you do the robust part? they chose to focus on spectrogram peaks which are consistent even in noisy environments
-   also a point about "approximate linear superposability" which idk what that means here exactly but idea is that net response of two stimuli is roughly equal to sum of individual responses to each stimulus
-   how to choose peaks? i think they define some "region" radius around a peak in time-frequency. and choose the value with highest score/amplitude. but they say this is different than energy content? idk why. energy content = uniform coverage(?) whereas amplitude survives noise
-   so the transformation is from a noisy spectogram to a "plot" of several points corresponding to peaks, without any amplitude, just a binary peak/no-point
-   by eliminating amplitude it becomes insensitive to EQ(???) which is desirable
-   the pattern of dots should be same (totally same?) for matching audio. i can't imagine it's perfectly the same? like there HAS to be frequency distortion
-   but overall i guess it makes sense. and they reduce the problem to sliding the dot-chart for a sample over various songs and collecting the various good matches
- they then talk about the speed of matching the dot-chart...
-   with a 1024-bin frequency y-axis that only gives you 10 bits of frequency data per peak? idk what this means lol. and somehow this makes it slow to match the chart? 


so my understanding now is
- fourier transform the song into spectrogram
- hash the song by taking only local maximum points within some time-freq range b/c it's resistant to noise, save it as f1,f2,t2-t1
- make database by hashing each second of a song and append the track id and total offset
- find song by hashing recorded music and searching each second-long chunk against db
-   how do you choose a good match? well if you have the same song, then you should have multiple consecutive chunks matching. for a given song in the db, you should be able to see that as time progresses in the recording, time progresses in the db song too, and you'll see a diagonal line of matches (x axis is db song time, y axis is sample time)
