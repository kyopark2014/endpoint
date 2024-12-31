# AWS Endpoint

(추후 다시 시도)

원래 계획은 [AWS PrivateLink를 통해 다른 리전의 Amazon Bedrock을 내부 네트워크에서 사용하기](https://aws.amazon.com/ko/blogs/tech/cross-region-bedrock-via-private-link/)을 참조해서 Private Subnet에 있는 EC2에 SSE로 접속 후 Endpoint 설치 전후로 바꾸어 테스트하려고 진행했으나 아래 문제가 있었습니다.

- EC2를 private subnet에 두고 security group의 allowAllOutbound를 false로 설정 SSE도 접속이 불가합니다.
- 이때, HTTPS (443)를 열면 SSE 접속은 되지만 aws cli도 HTTPS를 사용하므로 접속이 가능하였습니다. (Endpoint 테스트 불가)
- 추후 방안을 찾아서 동작 테스트 예정입니다.

<img width="800" alt="image" src="https://github.com/user-attachments/assets/157efbad-fe3a-415c-a3db-80789bb613ec" />


## 주요 설정

EC2의 Security Group은 아래와 같습니다. 

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

S3의 접속 확인을 위해 사용하는 명령어는 아래와 같습니다. 

```text
ls storage-for-endpoint-262976740991-us-west-2 --cli-connect-timeout 5
```

Bedrock에 대한 연결을 아래 명령어로 확인합니다. 

```text
aws bedrock list-foundation-models --region us-west-2 --cli-connect-timeout 5
```

접속이 안되는 경우는 아래와 같습니다.

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
