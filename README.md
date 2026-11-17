# Check the model ID your deployment expects

An OpenAI-compatible model catalog can respond with HTTP 200 while omitting the model your application needs. A smoke test can catch that mismatch before the deployment makes its first generation request.

This Ailoot AI engineering note was written with AI assistance. The example was checked against the Ailoot API reference and exercised with local HTTP fixtures.

## Check the exact ID

Set `AILOOT_API_KEY` to a workspace key using your existing secret store. Set `AILOOT_MODEL_ID` to the exact ID your application will request. Run the test from the same service, job or deployment environment that will use the API. It sends one `GET /v1/models` request and checks the returned IDs. [Ailoot model-list reference](https://ailoot.ai/docs/en/api/models/)

```python
import json
import os
from urllib.error import HTTPError, URLError
from urllib.request import HTTPRedirectHandler, Request, build_opener


class NoRedirect(HTTPRedirectHandler):
    def redirect_request(self, req, fp, code, msg, headers, newurl):
        return None


expected_model = os.environ["AILOOT_MODEL_ID"]
request = Request(
    "https://api.ailoot.ai/v1/models",
    headers={"Authorization": f"Bearer {os.environ['AILOOT_API_KEY']}"},
)

try:
    with build_opener(NoRedirect()).open(request, timeout=10) as response:
        catalog = json.load(response)
except HTTPError as exc:
    raise SystemExit(f"Catalog request returned HTTP {exc.code}") from None
except (URLError, TimeoutError):
    raise SystemExit("Catalog request did not complete") from None
except (UnicodeDecodeError, json.JSONDecodeError):
    raise SystemExit("Catalog response is not valid JSON") from None

models = catalog.get("data") if isinstance(catalog, dict) else None
if not isinstance(models, list):
    raise SystemExit("Catalog response has no model list")
if not any(isinstance(model, dict) and model.get("id") == expected_model for model in models):
    raise SystemExit("The expected model ID is missing from this catalog")

print("The catalog includes the expected model ID")
```

The probe stops on a redirect so it will not forward the workspace credential to another host. A failure message reports the status or failed check. Keep the key and complete authorization header out of CI logs. Record the provider configuration and expected model ID alongside the smoke-test fixture.

The catalog confirms visibility under that key. To check generation, run separately with one short request that exercises the feature your application uses, such as streaming or structured output. [First SDK smoke test on Medium](https://medium.com/@developer_67484/the-first-request-to-send-when-switching-an-openai-sdk-to-a-new-endpoint-8383f6a184ac) · [Compatibility reference](https://ailoot.ai/docs/en/compatibility/)
