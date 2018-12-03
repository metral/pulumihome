# Manually setting up billing for an Organization

Normally, billing information for an organization is created when the organization is created (since we now require billing information when creating an Organization on app.pulumi.com). However, in certain cases, you may need to manually create billing information for customer (a common case for this is if we onboard a SAML organization for them, which is still a manual process).

Doing this requires interacting with our Stripe account and Database directly.

You'll need to create a Stripe Customer and Subscription object in Stripe and then tie these values to our database.

1. Navigate to the Stripe Dashboard.  If you don't have access, Eric can get you access.
2. Ensure the "View Test Data" toggle is off.
3. Create a new customer.  The e-mail you put in will be who gets the invoices.  Description should be `<organization_name> organization`.  e.g. "tableau organization"
4. On the customer information page, add two pieces of metadata in the Metadata section:
    - `pulumi.orgID` - this should be set to the ID of the organization in our database.  You can get this value by running: `select id, github_login from Organizations WHERE github_login = "<org-name here>";`  e.g. `select id, github_login from Organizations WHERE github_login = "tableau";`
    - `pulumi.apiDomain` - this should be `api.pulumi.com`
5. Now, create a subscription object. Go to Billing -> Subscriptions and then click New Subscription. Pick the customer you just created, and the plan you want to tie them to. Set the number of stacks the organization should have.  Because we don't have a card on file, you'll have to use the "send invoice" option.  You can also put the plan on a trial.
6. After creating the subscription, set the `pulumi.orgID` and `pulumi.apiDomain` metadata on this object as well, as you did above.

Note the customer and subscription IDs for the objects you just created.  They should be of the form `cus_XXXXXXXXX` and `sub_XXXXXXXX`.

Now, we need to add a row to the subscriptions table.  To do so, you need to run this query against the production database:

```
INSERT INTO Subscriptions (org_id, stripe_customer_id, stripe_subscription_id) VALUES ('<org-id>', '<customer-id>', '<subscription-id>');
```

e.g.

```
INSERT INTO Subscriptions (org_id, stripe_customer_id, stripe_subscription_id) VALUES ('f987e218-9724-4efa-adbc-18b3143c0c73', 'cus_E5X2RWMNpYcLa1', 'sub_E5XAHElBhJOLGc');
```

Remember to post a message to `#ops` in Slack and get an ACK from on-call before running queries against the production database.

