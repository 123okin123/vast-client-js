# VAST Javascript Client

## Usage

``` javascript
DMVASTClient.get(VASTURL, function(response)
{
    if (response)
    {
        for (var adIdx = 0, adLen = VASTResponse.ads.length; adIdx < adLen; adIdx++)
        {
            var ad = VASTResponse.ads[adIdx];
            for (var creaIdx = 0, creaLen = ad.creatives.length; creaIdx < creaLen; creaIdx++)
            {
                var linearCreative = ad.creatives[creaIdx];
                if (linearCreative.type != "linear") continue;

                for (var mfIdx = 0, mfLen = linearCreative.mediaFiles.length; mfIdx < mfLen; mfIdx++)
                {
                    var mediaFile = linearCreative.mediaFiles[mfIdx];
                    if (mediaFile.mimeType != "video/mp4") continue;

                    player.vastTracker = new DMVASTTracker(ad, linearCreative);
                    player.vastTracker.on('clickthrough', function(url)
                    {
                        document.location.href = url;
                    });
                    player.on('canplay', function() {self.vastTracker.load();});
                    player.on('timeupdate', function() {self.vastTracker.setProgress(self.currentTime);});
                    player.on('play', function() {self.vastTracker.setPaused(false);});
                    player.on('pause', function() {self.vastTracker.setPaused(true);});

                    player.video.href = mediaFile.fileURL;
                    // put player in ad mode
                    break;
                }

                if (player.vastTracker)
                {
                    break;
                }
            }

            if (player.vastTracker)
            {
                break;
            }
            else
            {
                // Inform ad server we can't find suitable media file for this ad
                DMVASTUtil.track(ad.errorURLTemplates, {ERRORCODE: 403});
            }
        }
    }

    if (!self.vastTracker)
    {
        // No pre-roll, start video
    }

});
```