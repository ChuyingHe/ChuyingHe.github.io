[TOC]

# Commands
<table>
<tbody>
  <tr>
    <td>helm create myChart</td>
    <td>to create skeleton</td>
  </tr>
  <tr>
    <td>helm dependency update</td>
    <td>to add dependencies and lock the versions</td>
  </tr>
  <tr>
    <td>helm install myChartApp .</td>
    <td>deploy the app from <b>current directory</b></td>
  </tr>
  <tr>
    <td>helm uninstall myChartApp</td>
    <td>delete deployment/release</td>
  </tr>
  <tr>
    <td>helm list</td>
    <td>list all deployment</td>
  </tr>
  <tr>
    <td>helm pull xxx</td>
    <td>download a chart from a Repo</td>
  </tr>
  <tr>
    <td>helm rollback xxx</td>
    <td>roll back to the previous release</td>
  </tr>
  <tr>
    <td>helm status myChartApp</td>
    <td>check status of named release</td>
  </tr>
  <tr>
    <td>helm show</td>
    <td>show info of a chart</td>
  </tr>
<tr>
    <td>helm template myChartApp helm-directory > base/deployment.yaml</td>
    <td>extract the object definition from <b>Helm Chart</b> into <b>Kustomize's</b><code>/base/deployment.yaml</code></td>
  </tr>
</tbody>
</table>