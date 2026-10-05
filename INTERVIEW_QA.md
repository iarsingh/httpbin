# httpbin — interview questions and answers

[README](README.md) · [Project architecture](PROJECT_ARCHITECTURE.md)

Answers below use this repository’s files and implementation. They distinguish existing behavior from suggested extensions; source links let you verify each walkthrough.

## 1. What problem does httpbin address, and what can you demonstrate?

A fork of the Python and Flask HTTP request and response inspection service.

I would demonstrate the linked implementation or examples and distinguish that evidence from any planned production features. Start with [`README.md`](README.md).

## 2. How is this repository organized?

- [`httpbin/core.py`](httpbin/core.py): Implementation or supporting configuration.
- [`httpbin/helpers.py`](httpbin/helpers.py): Implementation or supporting configuration.
- [`httpbin/filters.py`](httpbin/filters.py): Implementation or supporting configuration.
- [`setup.py`](setup.py): Implementation or supporting configuration.
- [`httpbin/__init__.py`](httpbin/__init__.py): Implementation or supporting configuration.
- [`httpbin/structures.py`](httpbin/structures.py): Implementation or supporting configuration.
- [`httpbin/utils.py`](httpbin/utils.py): Implementation or supporting configuration.
- [`Dockerfile`](Dockerfile): Container build/service configuration.

[PROJECT_ARCHITECTURE.md](PROJECT_ARCHITECTURE.md) contains the component diagram and the implementation walkthrough.

## 3. Can you walk through `status_code` and explain the decision it makes?

The main walkthrough here is `status_code(code)` in [`httpbin/helpers.py`](httpbin/helpers.py#L207). Returns response object of given status code.

```python
def status_code(code):
    """Returns response object of given status code."""

    redirect = dict(headers=dict(location=REDIRECT_LOCATION))

    code_map = {
        301: redirect,
        302: redirect,
        303: redirect,
        304: dict(data=''),
        305: redirect,
        307: redirect,
        401: dict(headers={'WWW-Authenticate': 'Basic realm="Fake Realm"'}),
        402: dict(
            data='Fuck you, pay me!',
            headers={
                'x-more-info': 'http://vimeo.com/22053820'
            }
        ),
        406: dict(data=json.dumps({
```

This is an excerpt; follow the source link for the rest of the branches.

The implementation calls `dict`, `json.dumps`, `make_response`. In an interview, trace those calls in execution order using a fixture input.

## 4. What responsibility does `response` have?

`response(credentials, password, request)` is defined in [`httpbin/helpers.py`](httpbin/helpers.py#L311). Compile digest auth response

If the qop directive's value is "auth" or "auth-int" , then compute the response as follows:
   RESPONSE = MD5(HA1:nonce:nonceCount:clienNonce:qop:HA2)
Else if the qop directive is unspecified, then compute the response as follows:
   RESPONSE = MD5(HA1:nonce:HA2)

Arguments:
- `credentials`: credentials dict
- `password`: request user password
- `request`: request dict

Its return expressions include:

- `response`

It uses `H`, `HA1`, `HA1_value.encode`, `HA2`, `HA2_value.encode`, `ValueError`, `b':'.join`, `credentials.get`. This is the code path I would compare against the caller to explain responsibility boundaries.

## 5. What input validation and failure behavior are implemented?

Explicit failure paths include:

- `ValueError` in [`httpbin/helpers.py`](httpbin/helpers.py#L308).
- `ValueError('qop value are wrong')` in [`httpbin/helpers.py`](httpbin/helpers.py#L350).
- `ValueError('%s required' % k)` in [`httpbin/helpers.py`](httpbin/helpers.py#L303).
- `ValueError('%s required for response H' % k)` in [`httpbin/helpers.py`](httpbin/helpers.py#L342).

I would test both the condition that reaches each exception and the caller that translates it. An explicit raise does not mean every malformed input or dependency failure is handled.

## 6. Which test would you use to demonstrate correctness?

[`test_httpbin.py`](test_httpbin.py#L117) contains `test_index`:

```python
    def test_index(self):   
        response = self.app.get('/', headers={'User-Agent': 'test'})
        self.assertEqual(response.status_code, 200)
```

This is a concrete regression example from the repository. Its assertions establish that case; they do not establish behavior for every input or under production load.

## 7. What HTTP interface does the code expose?

- `ROUTE /legacy` → `view_landing_page` in [`httpbin/core.py`](httpbin/core.py#L241).
- `ROUTE /html` → `view_html_page` in [`httpbin/core.py`](httpbin/core.py#L247).
- `ROUTE /robots.txt` → `view_robots_page` in [`httpbin/core.py`](httpbin/core.py#L263).
- `ROUTE /deny` → `view_deny_page` in [`httpbin/core.py`](httpbin/core.py#L282).
- `ROUTE /ip` → `view_origin` in [`httpbin/core.py`](httpbin/core.py#L301).
- `ROUTE /uuid` → `view_uuid` in [`httpbin/core.py`](httpbin/core.py#L317).
- `ROUTE /headers` → `view_headers` in [`httpbin/core.py`](httpbin/core.py#L333).
- `ROUTE /user-agent` → `view_user_agent` in [`httpbin/core.py`](httpbin/core.py#L349).

These are literal decorators. Application/router prefixes, authentication, and middleware must be checked in the corresponding setup code.

## 8. Where does state live, and what happens with multiple workers?

Module-level containers include `app.config['SWAGGER']`, `template`, `swagger_config` in [`httpbin/core.py`](httpbin/core.py); `ACCEPTED_MEDIA_TYPES` in [`httpbin/helpers.py`](httpbin/helpers.py).

These containers belong to a Python process. Inspect which are constant fixtures and which are mutated. Mutable process state needs an explicit shared-storage or synchronization strategy before multiple workers can provide consistent behavior.

## 9. How would another engineer reproduce your walkthrough?

Follow [`README.md`](README.md) and the linked component documents. This documentation update does not assert an application launch command for a repository without a verified launch contract.

## 10. How would you add CI without confusing it with deployment?

First automate the repository-specific checks above, including documentation link validation. Add deployment only after defining the target environment, required credentials, approval boundary, smoke test, and rollback procedure. No GitHub Actions workflow is asserted by the inspected inventory.

## 11. How would you present this project in a Forward Deployed Engineer interview?

Start with the user and operational problem described in [`README.md`](README.md). Explain one constraint that changes the implementation, show the linked code or example, and walk through a success case and a failure case. Agree on a measurable acceptance criterion before expanding the solution, and leave a handoff with data boundaries and rollback ownership. Any proposed production or business metric should be identified as a target until measured.

## 12. What is the input-to-output contract of `status_code`?

In [`httpbin/helpers.py`](httpbin/helpers.py#L207), `status_code(code)` receives the inputs. The function computes these intermediate values:

- `redirect = dict(headers=dict(location=REDIRECT_LOCATION))`
- `code_map = {301: redirect, 302: redirect, 303: redirect, 304: dict(data=''), 305: redirect, 307: redirect, 401: dict(headers={'WWW-Authenticate': 'Basic realm="Fake Realm"'}), 402: dict(data='Fuck you, pay me!', headers={'x-more-info': 'http://vimeo.com/22053820'}), 406: dict(data=json.dumps({'message': 'Client did not request a supported media type.', 'accept': ACCEPTED_MEDIA_TYPES}), headers={'Content-Type': 'application/json'}), 407: dict(head`
- `r = make_response()`
- `r.status_code = code`

Its result is defined by:

- `r`

## 13. Which decision rules or boundary conditions should an interviewer challenge?

The implementation in [`httpbin/helpers.py`](httpbin/helpers.py#L207) branches on:

- `code in code_map`
- `'data' in m`
- `'headers' in m`

A useful extension is a table-driven test that covers each condition just below, at, and above its boundary where applicable. These expressions are the current rules; changing them changes behavior and should be justified by the project’s acceptance criteria.
