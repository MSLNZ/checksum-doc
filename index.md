# Documentation
Requesting the status of one or more MD5 checksums will return a JSON object where the keys are the requested checksums and the corresponding value is one of: `valid`, `cancelled` or `unrecognised`.

You can submit a request by either using a [query string](https://developer.mozilla.org/en-US/docs/Web/API/URL/search) or as a JSON array in the [body message](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages). For a query string, the service has an approximate 10,000 character limit for the length of the URL string. This limit corresponds to approximately 300 checksum values. Your web browser, or the application you use to generate the request URL, may have a smaller limit for the maximum number of characters it supports. The body message has essentially no limit to the number of checksums that can be sent in a single request.

### Query String

To get the status of a checksum (e.g., `4123d4000116b9f48dd18a6313382d5b`) you could use the [curl] command-line tool to send the following request

```console
curl https://kapua.measurement.govt.nz/checksum?4123d4000116b9f48dd18a6313382d5b
```

which may return the following response

```console
{
  "4123d4000116b9f48dd18a6313382d5b": "valid"
}
```

Alternatively, you could enter the URL in a web browser to view the response

[https://kapua.measurement.govt.nz/checksum?4123d4000116b9f48dd18a6313382d5b](https://kapua.measurement.govt.nz/checksum?4123d4000116b9f48dd18a6313382d5b)

You can specify multiple checksums by separating each checksum value by a comma. For example, using [curl]

```console
curl https://kapua.measurement.govt.nz/checksum?4123d4000116b9f48dd18a6313382d5b,34c06fa135f07c95838fd18a9b6e273f,kiwi
```

or view the response in a web browser

[https://kapua.measurement.govt.nz/checksum?4123d4000116b9f48dd18a6313382d5b,34c06fa135f07c95838fd18a9b6e273f,kiwi](https://kapua.measurement.govt.nz/checksum?4123d4000116b9f48dd18a6313382d5b,34c06fa135f07c95838fd18a9b6e273f,kiwi)

### Body Message

You can either send a `POST` or a `GET` request to the server (these HTTP methods are treated equivalently by the server). The body of the message must be a JSON array of checksum values.

The following example uses the [curl] command-line tool

```console
curl -H 'Content-Type: application/json' -d '["4123d4000116b9f48dd18a6313382d5b","34c06fa135f07c95838fd18a9b6e273f"]' https://kapua.measurement.govt.nz/checksum
```

which may return the following response

```console
{
  "4123d4000116b9f48dd18a6313382d5b": "valid",
  "34c06fa135f07c95838fd18a9b6e273f": "cancelled"
}
```

#### Invalid requests

If you specify an invalid JSON array, such as the following _(note the extra trailing comma after the second checksum value)_,

```console
curl -H 'Content-Type: application/json' -d '["4123d4000116b9f48dd18a6313382d5b","34c06fa135f07c95838fd18a9b6e273f",]' https://kapua.measurement.govt.nz/checksum
```

a query string that has too many characters, or an invalid URL route, the [status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) of the response will be one of the `4xx`-series status codes (instead of `200`) and the content of the response will be a JSON object with a `"message"` key that gives the reason why the request failed. For the previous invalid JSON-array request, the returned status code would be `400` and content of the response would be

```console
{"message": "Error! Invalid JSON object: '[\"4123d4000116b9f48dd18a6313382d5b\",\"34c06fa135f07c95838fd18a9b6e273f\",]'"}
```

### Python Example

The following example shows how to generate an MD5 checksum from a file and then query the service for the status of that checksum

```python
import hashlib
import json
from pathlib import Path
from urllib import request

# Get the MD5 checksum of a file (replace "UPDATE.me" with the path to your file)
c = hashlib.md5(Path("UPDATE.me").read_bytes()).hexdigest()

# Query the status of the checksum
with request.urlopen(f"https://kapua.measurement.govt.nz/checksum?{c}") as r:
    response = json.loads(r.read())
    print(response)
```

[curl]: https://curl.se/
