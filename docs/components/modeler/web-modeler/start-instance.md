---
id: start-instance
title: Start a new process instance
description: "If a BPMN diagram is deployed via Web Modeler, you can start a new instance of this diagram."
---

<span class="badge badge--cloud">Camunda Cloud only</span>

If a BPMN diagram is deployed via Web Modeler, you can start a new instance of this diagram.

To do so, click **Start Instance**:

![start instance](img/web-modeler-start-instance-modal-healthy.png)

You can also specify variables written to the process context at startup. The variables must be formatted in valid JSON. As an example, you can use the following JSON:

```json
{
  "hello": "world"
}
```