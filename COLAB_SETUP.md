## Colab session checklist

1. **Runtime:** use CPU for data exploration; switch to a GPU (Runtime → Change runtime type) only when training.
2. **Mount Drive:** folder icon in the left sidebar → "Mount Drive".
3. **Copy and unzip the data to local disk** (much faster than reading from Drive):
   ```
   !cp /content/drive/MyDrive/zib-project/data/ts_small.zip /content/
   !unzip -q /content/ts_small.zip -d /content/data
   !rm /content/ts_small.zip
   ```
4. **Check that it worked:**
   ```
   !ls /content/data | head
   !df -h /content
   ```
5. **Get the latest code** (if the repo isn't in `/content` yet, use `git clone` instead of `git pull`):
   ```
   !git -C /content/<repo> pull
   ```
6. **Before closing:** save anything you want to keep to `/content/drive/MyDrive/zib-project/outputs/`. Everything else in `/content` is deleted when the runtime disconnects.