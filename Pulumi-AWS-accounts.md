| Account name | ID | Log in |
| --- | --- | --- |
| (dev) | 153052954103 | n/a |
| TestCustomer | 678725633224 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=678725633224&displayName=TestCustomer)
| learningmachine-malta | 977316828113 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=977316828113&displayName=learningmachine-malta)
| learningmachine-prod | 999780342027 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=999780342027&displayName=learningmachine-prod)
| learningmachine-stage | 127455204268 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=127455204268&displayName=learningmachine-stage)
| logs-vault | 031982275711 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=031982275711&displayName=logs-vault)
| ppc-production-learningmachine-auto | 750427423503 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=750427423503&displayName=ppc-production-learningmachine-auto)
| ppc-production-learningmachine-malta | 894273677487 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=894273677487&displayName=ppc-production-learningmachine-malta)
| ppc-production-learningmachine-prod | 396386917111 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=396386917111&displayName=ppc-production-learningmachine-prod)
| ppc-production-learningmachine-stage | 394069743421 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=394069743421&displayName=ppc-production-learningmachine-stage)
| ppc-production-moolumi-default | 502054765176 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=502054765176&displayName=ppc-production-moolumi-default)
| ppc-production-pulumi-default | 298907078223 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=298907078223&displayName=ppc-production-pulumi-default)
| ppc-staging-moolumi-default | 269812760766 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=269812760766&displayName=ppc-staging-moolumi-default)
| ppc-staging-pulumi-default | 268538227356 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=268538227356&displayName=ppc-staging-pulumi-default)
| ppc-testing-moolumi-default | 873203032246 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=873203032246&displayName=ppc-testing-moolumi-default)
| ppc-testing-pulumi-default | 359955403598 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=359955403598&displayName=ppc-testing-pulumi-default)
| pulumi-production | 058607598222 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=058607598222&displayName=pulumi-production)
| pulumi-staging | 098437015098 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=098437015098&displayName=pulumi-staging)
| pulumi-testing | 086028354146 | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=086028354146&displayName=pulumi-testing)


generated with:
```python
import boto3
import string

organizations = boto3.client("organizations")
accounts = organizations.list_accounts()["Accounts"]
accounts = [a for a in accounts if a["Id"] != "153052954103"]

t = string.Template("| ${Name} | ${Id} | [AWS console](https://signin.aws.amazon.com/switchrole?roleName=OrganizationAccountAccessRole&account=${Id}&displayName=${Name})")

print("| Account name | ID | Log in |")
print("| --- | --- | --- |")
print("| (dev) | 153052954103 | n/a |")

for account in sorted(accounts, key=lambda x: x["Name"]):
    print(t.substitute(**account))
```