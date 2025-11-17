# setup-cloudflare-warp
![Tests](https://github.com/Boostport/setup-cloudflare-warp/actions/workflows/tests.yml/badge.svg)

The `Boostport/setup-cloudflare-warp` action sets up Cloudflare WARP in your GitHub Actions workflow. It allows GitHub
Actions workflows to access resources that are secured by Cloudflare Zero Trust.

## Usage
This action currently only supports Linux, macOS and Windows. Contributions to support Microsoft Windows are welcome.

To use this action, generate a service token using these
[instructions](https://developers.cloudflare.com/cloudflare-one/identity/service-tokens/) and configure the action:

Example:
```yaml
uses: Boostport/setup-cloudflare-warp@v1
with:
  organization: your-organization
  auth_client_id: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_ID }}
  auth_client_secret: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_SECRET }}
```
You can specify the version of Cloudflare WARP to install:
```yaml
uses: Boostport/setup-cloudflare-warp@v1
with:
  version: 2023.1.133
  organization: your-organization
  auth_client_id: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_ID }}
  auth_client_secret: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_SECRET }}
```

You can also specify a unique client identifier for the device:
```yaml
uses: Boostport/setup-cloudflare-warp@v1
with:
  organization: your-organization
  auth_client_id: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_ID }}
  auth_client_secret: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_SECRET }}
  unique_client_id: bc6ea6f6-a7c9-4da0-b303-69f5481803b8
```

You can also specify virtual network you want to use:
```yaml
uses: Boostport/setup-cloudflare-warp@v1
with:
  organization: your-organization
  auth_client_id: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_ID }}
  auth_client_secret: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_SECRET }}
  vnet: ${{ secrets.CLOUDFLARE_VNET }}
```

You can configure connection stability checking (enabled by default) to ensure WARP is fully ready before proceeding:
```yaml
uses: Boostport/setup-cloudflare-warp@v1
with:
  organization: your-organization
  auth_client_id: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_ID }}
  auth_client_secret: ${{ secrets.CLOUDFLARE_AUTH_CLIENT_SECRET }}
  connection_stability_check: true
  stability_check_endpoint: https://your-internal-service.example.com/health
```

## Inputs
- `version` - (optional) The version of Cloudflare WARP to install. Defaults to the latest version.
- `organization` - (required) The name of your Cloudflare Zero Trust organization.
- `auth_client_id` - (required) The service token client id.
- `auth_client_secret` - (required) The service token client secret.
- `unique_client_id` - (optional) A unique identifier for the client device. See [Cloudflare documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/deployment/mdm-deployment/parameters/#unique_client_id) for more details.
- `configure_docker_dns` - (optional) *Linux Only* Configure Docker to use Cloudflare WARP for DNS resolution. Defaults to `false`.
- `vnet` - (optional) Virtual network ID
- `connection_stability_check` - (optional) Enable connection stability check that verifies DNS resolution and connectivity. This ensures WARP is fully ready before proceeding. Defaults to `true`. Set to `false` to disable.
- `stability_check_endpoint` - (optional) Custom endpoint URL to test connectivity against. If not provided, uses default Cloudflare endpoint (`https://www.cloudflare.com/cdn-cgi/trace`). Useful for testing connectivity to your internal services.

## Cloudflare Permissions
> [!TIP]
> Failure to set the proper permission will result in a `Status update: Unable to connect. Reason: Registration Missing` error.

Under `Zero Trust > Settings > WARP Client > Device enrollment permissions` a policies rule must have `SERVICE AUTH` set as the rule action.
![Cloudflare Device Enrollment Policy](./docs/resources/cloudflare_device_enrollment.png)

To add the GitHub action to a WARP Client Profile, you must specify the expression of the policy to `User Email`, `is`, `non_identity@<INSERT YOUR ORG>.cloudflareaccess.com`.


## Connection Stability

By default, the action performs a connection stability check after WARP connects. This verifies:
- DNS resolution is working through WARP
- Network connectivity is established (tests against Cloudflare's endpoint or your custom endpoint)

This ensures the connection is fully ready before your workflow continues, eliminating the need for manual sleep delays. The stability check uses exponential backoff with up to 60 attempts (approximately 5-10 minutes maximum wait time).

If you experience connection issues, you can:
- Enable stability checking (default): The action will wait until connectivity is verified
- Specify a custom endpoint: Test against your actual service endpoint to ensure it's accessible
- Disable stability checking: Set `connection_stability_check: false` if you prefer to handle verification manually

## Troubleshooting
- Unable to connect: `Status update: Unable to connect. Reason: Registration Missing` errors
  - Check that the service token is valid and not expired.
  - Check that the service token has the appropriate permissions to connect.
  - Cancel and restart the job, sometimes there's an issue on Cloudflare's end that causes this error.
- Connection appears connected but requests fail
  - Enable `connection_stability_check` (default) to ensure the connection is fully ready
  - Specify `stability_check_endpoint` to test against your actual service endpoint
  - Check that DNS resolution is working correctly

## Disclaimer
This is not an official Cloudflare product nor is it endorsed by Cloudflare.
