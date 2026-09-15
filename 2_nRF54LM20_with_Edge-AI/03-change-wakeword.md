# Change wake-word

**Download model after training complete and follow README.md for deployment guide:**

Replace `"src/nrf_edgeai_generated"` folder in the sample with your model folder from the downloaded archive.

**Update the model name in the application code:**

In src/wakeword.c update the following line with the new model name from your nrf_edgeai_user_model.h:

```c 
ww_model = nrf_edgeai_user_model_wakeword();
```

Fine-tune the prediction postprocessing logic if needed using Kconfig options: `WW_PROBABILITY_THRESHOLD` and `WW_COUNT_THRESHOLD`.


## Already created WAKE-WORDS
To save time in the folder `custom-wake-word` you find four already trained keywords.

| Wake Word | Folder                                         |
|-----------|------------------------------------------------|
| wake up   | FAE_Sales Conference (wake up)_95558_wake_word |
| avnet     | Wake_Word - Avnet_95575_wake_word              |
| future    | Wake_Word - Future_95576_wake_word             |
| rutronik  | Wake_Word - Rutronik_95574_wake_word           |
