# AWS ImageBuilderサンプル

- Component用のyamlをおくS3バケット作る
```bash
$ aws s3 mb s3://megun-imagebuilder-samples
$ aws s3api put-public-access-block --bucket megun-imagebuilder-samples --public-access-block-configuration "BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true"
```
- s3にcomponent用yamlをおく
```bash
$ aws s3 cp component.yaml s3://megun-imagebuilder-samples/component.yaml
```

- component作る
```bash
$ aws imagebuilder create-component --cli-input-json file://create-component.json
```

- recipe作る
```bash
$ aws imagebuilder create-image-recipe --cli-input-json file://create-image-recipe.json
```

- distribution-config作成
```bash
$ aws imagebuilder create-distribution-configuration --cli-input-json file://create-distribution-configuration.json
```

- ingrastructure-config作成
```bash
$ aws imagebuilder create-infrastructure-configuration --cli-input-json file://create-infrastructure-configuration.json
```

- pipeline作成
```bash
aws imagebuilder create-image-pipeline --cli-input-json file://create-image-pipeline.json
```