# API Gateway + Lambda Static Site Deployment

This deployment serves static files over HTTPS from API Gateway and Lambda, while storing site content in a private S3 bucket. It also provisions a DynamoDB table and API routes for custom tunings.

## Deploy stack

1. Create a stack using `apigw_lambda_template.yaml`.
2. Keep default parameters unless you need custom names.
3. If your lab account blocks `iam:CreateRole`, set `ExistingLambdaExecutionRoleArn` to a pre-provisioned role ARN (for example, a role provided by the lab).

### Minimum permissions needed on existing Lambda role

The existing role should allow:

- `logs:CreateLogGroup`
- `logs:CreateLogStream`
- `logs:PutLogEvents`
- `s3:GetObject` on the stack bucket objects
- `dynamodb:PutItem`
- `dynamodb:GetItem`
- `dynamodb:UpdateItem`
- `dynamodb:DeleteItem`
- `dynamodb:Scan` on the tunings table

## Upload tuner.html

After stack creation, upload your current `tuner.html` to the output bucket:

```powershell
aws s3 cp tuner.html s3://<BucketNameOutput>/tuner.html --content-type "text/html; charset=utf-8"
```

## Open site

Use the stack output `WebsiteURL`.

## Tunings API (no user accounts)

Use stack output `ApiBaseURL` for the base endpoint.

- `GET /api/tunings` list tunings
- `POST /api/tunings` create tuning
- `GET /api/tunings/{tuningId}` fetch one tuning
- `PUT /api/tunings/{tuningId}` update tuning
- `DELETE /api/tunings/{tuningId}` delete tuning

Create payload format:

```json
{
	"name": "My 7-note scale",
	"description": "Optional notes",
	"data": {
		"notes": ["C", "D", "E", "F", "G", "A", "B"],
		"referenceHz": 440
	}
}
```

## Notes

- The URL is HTTPS by default (API Gateway), so browser microphone APIs can work.
- The S3 bucket is private and not exposed as a website endpoint.
- You can add files later (for example `styles.css`, `app.js`) and they will be served through `GET /{proxy+}`.
- CORS is enabled for API calls so browser requests from the hosted frontend can call `/api/tunings`.
