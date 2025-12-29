# Clock Learning App - Cloud Run Deployment

## Deploy via Cloud Shell (Recommended)

1. Go to https://console.cloud.google.com
2. Open **Cloud Shell** (terminal icon in top right)
3. Upload this zip file (three-dot menu → Upload)
4. Run these commands:

```bash
unzip clock-cloudrun.zip
cd clock-cloudrun
gcloud run deploy clock-learning \
  --source . \
  --region us-central1 \
  --allow-unauthenticated
```

5. You'll get a URL like: `https://clock-learning-xxxxx-uc.a.run.app`

## Map to a Subdomain (Optional)

To use a custom domain like `clock.vahana.vc`:

1. Go to Cloud Run in the Google Cloud Console
2. Select your service → Domain Mappings → Add Mapping
3. Enter your subdomain and follow DNS verification steps

## Updating

After making changes to `index.html`, run:
```bash
gcloud run deploy clock-learning --source . --region us-central1 --allow-unauthenticated
```
