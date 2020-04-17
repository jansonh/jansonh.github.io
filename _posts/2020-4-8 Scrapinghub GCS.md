---
layout: post
title: Scrapy Cloud - Storing Results in Google Cloud Storage
comments: true
---

While building my pet project, I found the need to scrape files from certain websites. Using scrapy library in Python, I wanted to download all files in the website pages to [Google Cloud Storage](https://cloud.google.com/products/storage) (GCS). 

The [scrapy documentation](https://docs.scrapy.org/en/latest/topics/media-pipeline.html) and [scrapinghub](https://support.scrapinghub.com/support/solutions/articles/22000225178-downloading-and-processing-images) is not really clear about the steps to authenticate the scrapy cloud to upload scrapped files to the GCS bucket. I tried to follow the tutorial [here](https://medium.com/@acowpy/scraping-files-images-using-scrapy-scrapinghub-and-google-cloud-storage-c7da9f9ac302) but to no success.

Here are the steps I follow to succesfully connect scrapy cloud and GCS:

1. Set up the GCS bucket and take note of the bucket name.
2. In your scrapy project, open the `settings.py` and add the `FilesPipeline`.

        ITEM_PIPELINES = {
            'scrapy.pipelines.files.FilesPipeline': 1,
        }

3. Specify your bucket name and Google Cloud Project ID in `settings.py`.

        FILES_STORE = 'gs://bucket-name/'
        GCS_PROJECT_ID = 'project-id'

    **Tip:** Don't forget to add trailing slash (directory separator) in the bucket name in `FILES_STORE`.
4. Next, you have to obtain the credentials to connect to GCS from your scrapy project. [Create the service account key here](https://console.cloud.google.com/apis/credentials/serviceaccountkey) by selecting *New service account* and JSON key type. From the role list, select *Project* > *Owner*. A JSON file will download to your computer. [See this page for more instructions](https://cloud.google.com/docs/authentication/production#obtaining_and_providing_service_account_credentials_manually).
5. In the same directory as your `settings.py`, find and open `__init__.py` and add this code [(source)](https://medium.com/@rutger_93697/i-thought-this-solution-was-somewhat-complex-3e8bc91f83f8).

        import os
        import json
        import pkgutil
        import logging

        path = "{}/google-cloud-storage-credentials.json".format(os.getcwd())

        credentials_content = '''
            escaped content of the credentials JSON
        '''

        with open(path, "w") as text_file:
            text_file.write(credentials_content)

        logging.warning("Path to credentials: %s" % path)
        os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = path

6. Open the JSON file and copy paste all the contents to [this site here](https://www.freeformatter.com/json-escape.html) to escape the string. Then copy the string to the `credentials content` in the above code.
7. Then in the spider code, returns a dict with the URL's key (`file_urls`). See the [scrapy documentation](https://docs.scrapy.org/en/latest/topics/media-pipeline.html#usage-example) for more detail.