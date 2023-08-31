---
title: "FluxのHelmReleaseでPostInstallがうまく実行されないのはFluxが--waitフラグを付けてるから"
date: 2023-08-31T19:30:00+09:00
draft: false
---

メモです。

## マイグレーション付きHelm ChartがFlux v2上でうまく動かない

例えばこれ。

https://github.com/minio/minio/blob/master/helm/minio/templates/post-job.yaml

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ template "minio.fullname" . }}-post-job
  labels:
    app: {{ template "minio.name" . }}-post-job
    chart: {{ template "minio.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-delete-policy": hook-succeeded,before-hook-creation
    {{- with .Values.postJob.annotations }}
      {{- toYaml . | nindent 4 }}
    {{- end }}
```

マイグレーションのような、メインのDeploymentから依存されている処理をHelmの `post-install` で行うのはよく見られます。ただ、これらのChartをFluxのHelmChartで管理しようとするとうまく動作しなくなることがあります。

FluxのHelmChartでは、デプロイに `helm install --wait` 相当の動作をしています。すると `"helm.sh/hook": post-install` なJobはすべてのDeploymentがRunningになるまで実行が行われません。しかし、上記のマイグレーションのような、それ自体が他のDeploymentに依存されている処理を行っている場合には、Deploymentは正常に起動せず、結果としてJobも実行されないというデッドロック状態に陥ってしまいます。

これを変更するにはHelmReleaseに `disableWait` を追記します。

https://fluxcd.io/flux/components/helm/helmreleases/#disabling-resource-waiting

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: minio
  namespace: minio-anyfrog
spec:
  chart:
    spec:
      chart: minio
      sourceRef:
        kind: HelmRepository
        name: minio
      version: 5.0.13
  interval: 1h0m0s
  valuesFrom:
  - kind: ConfigMap
    name: minio-config
    valuesKey: values.yaml
  install:
    disableWait: true # here
  upgrade:
    disableWait: true # here
```

これを指定することで、FluxがHelm installやupgradeの完了を曖昧にしか判断出来なくなるリスクも存在するため注意が必要です。

というか、 `--wait` の有無で動作が変わり過ぎでは...。 `pre-install` と `post-install` の中間のHookが作られればいい感じだとおもうんですが、そうやらないのは何か理由があるんですかね。

# 参考

[flux HelmRelease not launching Jobs equivalently to `helm install` · fluxcd/flux2 · Discussion #1085](https://github.com/fluxcd/flux2/discussions/1085)

