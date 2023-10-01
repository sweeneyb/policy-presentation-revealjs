---
author: Brian Sweeney
title: Policy For Use-Case Awareness
date: Oct 5, 2023
---

## How Did We Get Here? 

And this is some text

* You're a multitenant platform SRE
* You provide a range of features
* Power user wants a narrow "happy path"
* How can you let them know when somebody goes "off path"?


---

# Let's Talk About Policy!
* if/then in code for one user is ... submoptimal
* My team should get out of the way of their data collection
* If only there were a language that would allow them to express rules (and maybe let us enforce them)


There is - it's Rego

---

## What's Rego

* Rego is the language behind Open Policy Agent and conftest.
* "Rego was inspired .. [a] query language. Rego queries are assertions on data stored in OPA. These queries can be used to define policies that enumerate instances of data that violate the expected state of the system”.[^1]
* So if your API produces JSON, we can see what does (and does not) match your policy.

[^1]: https://www.openpolicyagent.org/docs/latest/policy-language

## How is this Useful

* Say your platform links workflows to some owner by their email address
* Did you validate the email is in internal? Is the email owned by a Managed Service Provider?

---

## Let's validate email domains
### (the data)
```
name: Second
size: 72
accountFlows:
  - act_flow_001:
    responsibleEmail: vhhcjfjl@sharklasers.com
    FlowNodes:
      - name: Flow_001
        address: flow001.vcap.me
        region: US
        parallelism: 5
      - name: Flow_002
        address: flow002.vcap.me
        region: US
        parallelism: 5
  - act_flow_002:
    responsibleEmail: vhhcjfjj@sharklasers.com
  - act_flow_003:
    responsibleEmail: vhhcjfjj@vcap.me
```
---

## Let's validate email domains
### the policy
```Rego
deny[msg] {
    flow := input.accountFlows[_]
    # print(flow)
    some key
    m := is_internal_email(flow[key].responsibleEmail)
    not m
    # print(flow)
    msg = sprintf("External email addres in flow  %v: %v", [key, flow[key].responsibleEmail])
}

is_internal_email(email_address) = true {
    pattern := "@sharklasers.com$"
    matched := regex.match(pattern, email_address)
    matched == true 
} else = false 
```
---

## Let's validate email domains
### the results
```
>conftest test --policy policies\domain \
backend/data/tenants/Second.yaml
FAIL - backend/data/tenants/Second.yaml - main - \
External email addres in flow act_flow_003: \
vhhcjfjj@vcap.me
1 test, 0 passed, 0 warnings, 1 failure, 0 exceptions
```
---

## But Biz Shouldn't Need conftest
* While userful, this is a tall ask for less technical folks
* Clould we "cloud" it?
* OPA is the engine running as a server
---

## OPA is a Decision Server[^2]
* Common infra for policy management/evaluation
* Works for one-offs and/or your prod flows
* Battle hardened, flexible deployments

[^2]: https://www.openpolicyagent.org/docs/latest/rest-api/

---

## A query With No Data on Disk
```bash
curl http://localhost:8000/tenant/Second  |\
  jq {"input":.} |\
  curl "http://localhost:8181/v1/data/main" \
    -H "Content-Type: application/json" --data @-
```
---

## But there's more, right?
* With our framework in place, can we do more interesting things?
::: incremental
* Let's write a rule based on business unit
* This rule will build on our existing ruleset

---

## A Rule on a Rule
### Data from an external source
* OPA can load static data or reach out to endpoints
```
{ "businessGroup": 
    { "HR": ["flow101.vcap.me", "flow001.vcap.me"],
      "engineering": ["flow101.vcap.me", "flow001.vcap.me"]
    }
}
```
---

## A Rule on a Rule
### HR email address policy
```
HRviolation[msg] {
    flow := input.accountFlows[_]
    some key
    m := is_internal_email(flow[key].responsibleEmail)
    not m

    some nodeSet
    flow[key].FlowNodes[nodeSet]["address"] in data.servers["businessGroup"]["HR"]
    msg = sprintf("External email addres in an HR flow  %v: %v", [key, flow[key].responsibleEmail])
}
```
## Add Feedback to API
```
app.post('/tenantWithCheck/:tenantId', async (req: Request, res: Response) => {
  let object: Tenant = req.body
  object['name'] = req.params.tenantId
  const result = await axios.post('http://localhost:8181/v1/data/main', {"input": object });
  console.log(result.data)
  if (result.data['result']['deny'].length > 0) {
    writeDoc(object)
    res.send(result.data['result']['deny']);
  } else {
    writeDoc(object)
    res.status(200).end()
  }
});
```

---

## Thanks!
* Demo?
* https://github.com/sweeneyb/policy-demos
