# endpoint
It shows how to deploy endpoints for security in AWS.

EC2를 private subnet에 두고 security group의 allowAllOutbound를 false로 설정하면 외부로 트래픽이 전송되지 않습니다.

```java
const ec2Sg = new ec2.SecurityGroup(this, `ec2-sg-for-${projectName}`,
  {
    vpc: vpc,
    allowAllOutbound: false,
    description: "Security group for ec2",
    securityGroupName: `ec2-sg-for-${projectName}`,
  }
);
```

이후 아래와 같이 명렁어를 입력하면 S3에 대한 연결을 확인할 수 있습니다. 이때, 외부로 접속이 안되므로 일정시간이 지나면 실패합니다. 

```text
ls storage-for-endpoint-262976740991-us-west-2 --cli-connect-timeout 5
```

마찬가지로 Bedrock에 대한 연결을 아래 명령어로 확인합니다. 마찬가지로 연결이 되지 않습니다.

```text
aws bedrock list-foundation-models --region us-west-2 --cli-connect-timeout 5
```

각 테스트 결과는 아래와 같습니다.

![image](https://github.com/user-attachments/assets/8e88961c-d244-447e-b98f-d2692b44f941)

[cdk-endpoint-stack.ts](./cdk-endpoint/cdk-endpoint-stack.ts)에서 아래의 주석된 부분을 풀어서 S3와 Bedrock에 대한 endpoint를 설치할 준비를 합니다. 

```text
// s3 endpoint
// const s3BucketAcessPoint = vpc.addGatewayEndpoint(`s3Endpoint-${projectName}`, {
//   service: ec2.GatewayVpcEndpointAwsService.S3,
// });

// s3BucketAcessPoint.addToPolicy(
//   new iam.PolicyStatement({
//     principals: [new iam.AnyPrincipal()],
//     actions: ['s3:*'],
//     resources: ['*'],
//   }),
// ); 

// Bedrock endpoint
// new ec2.InterfaceVpcEndpoint(this, `VPC Endpoint-${projectName}`, {
//   privateDnsEnabled: true,
//   vpc: vpc,
//   service: new ec2.InterfaceVpcEndpointService('com.amazonaws.us-west-2.bedrock', 443),
//   subnets: {
//     subnetType: ec2.SubnetType.PRIVATE_ISOLATED
//   }
// });
```

아래 명령어로 endpoint을 설치합니다. 

```text
cdk deploy --all
```

이후 
