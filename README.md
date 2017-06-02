```
gsutil mb gs://metadata_experiments/
gsutil rsync metadata_samples gs://metadata_experiments/metadata_samples
# gsutil ls gs://metadata_experiments/metadata_samples
# You can't grant access to a folder, so just make the whole bucket public for now
# gsutil acl ch -u AllUsers:R gs://metadata_experiments/metadata_samples
# https://cloud.google.com/storage/docs/access-control/making-data-public
gsutil iam ch allUsers:objectViewer gs://metadata_experiments
curl -s -D - https://metadata_experiments.storage.googleapis.com/metadata_samples/plain.txt?ignoreCache=1
# View metadata on the gzip.txt file
# gsutil ls -L gs://metadata_experiments/metadata_samples/gzip.txt
# https://cloud.google.com/storage/docs/transcoding
gsutil setmeta -h "Content-Encoding:gzip" gs://metadata_experiments/metadata_samples/gzip.txt
# gsutil ls -L gs://metadata_experiments/metadata_samples/gzip.txt
# Without accept-encoding
curl \
  -s -D - \
  https://metadata_experiments.storage.googleapis.com/metadata_samples/gzip.txt?ignoreCache=1
# With it!
curl \
  -H "Accept-Encoding: gzip" \
  -s -D - \
  https://metadata_experiments.storage.googleapis.com/metadata_samples/gzip.txt?ignoreCache=1
# Cloud Storage allows you to set CORS configuration at the bucket level only
# https://cloud.google.com/storage/docs/cross-origin
cat << EOF > cors-json-file.json
[
    {
      "origin": ["*"],
      "responseHeader": ["Content-Type"],
      "method": ["GET", "HEAD", "DELETE"],
      "maxAgeSeconds": 3600
    }
]
EOF
gsutil cors set cors-json-file.json gs://metadata_experiments
# Need ?ignoreCache=1 otherwise you'll be using the edge cache with cached metadata
# Preflight should contain `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, and `Access-Control-Allow-Headers`
curl \
  -H "Origin: http://localhost:8000" \
  -H "Access-Control-Request-Method: GET" \
  -X OPTIONS \
  -s -D - \
  https://metadata_experiments.storage.googleapis.com/metadata_samples/plain.txt?ignoreCache=1
# Actual request
curl \
  -H "Origin: http://localhost:8000" \
  -s -D - \
  https://metadata_experiments.storage.googleapis.com/metadata_samples/plain.txt?ignoreCache=1
# CORS and GZip decompressed
curl \
  -H "Origin: http://localhost:8000" \
  -s -D - \
  https://metadata_experiments.storage.googleapis.com/metadata_samples/gzip.txt?ignoreCache=1
# CORS and GZip with Content-Encoding
curl \
  -H "Accept-Encoding: gzip" \
  -H "Origin: http://localhost:8000" \
  -s -D - \
  https://metadata_experiments.storage.googleapis.com/metadata_samples/gzip.txt?ignoreCache=1
```
