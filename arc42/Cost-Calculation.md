# Cost Calculations

## Cost per Device

In order to have a sense what leading drivers for cost are, we explored the AWS Cost Calculator and configured for the cost of having exactly one powermate device. To be noted, this specific cost calculation only factors the *backend side* cost of having exactly one powermate device. Costs incurred due to for example the users usage of the powermate app (e.g. the AWS API Gateway) are not included in this specific cost calculation.

The per-device cost is composited as follows:

- IoT Core: 4.17 USD per month
- SQS: 0.67 USD per month
- AWS Lambda: 0.11 USD per month
- Amazon Timestream: 0.65 per month

**Total**: 5.60 USD per Month

For more details, see [cost estimate per device](cost-estimates/cost-per-device.json)
