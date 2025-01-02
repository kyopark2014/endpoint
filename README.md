# AWS Endpoint

[Endpoint 확인 방법](https://github.com/kyopark2014/llm-streamlit?tab=readme-ov-file#endpoint-%ED%99%95%EC%9D%B8-%EB%B0%A9%EB%B2%95)와 같이 NAT를 넣은 상태에서 동작학인 후에 NAT를 0으로 하면, endpoint를 제외한 모든 외부 접속이 끊겨서 Bedrock와 S3에 대한 테스트를 완료할 수 있었습니다.

SSE로는 아래 테스트가 안되는것으로 보여집니다. (2025.1.3)

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

// const bedrockEndpoint = vpc.addInterfaceEndpoint(`bedrock-endpoint-${projectName}`, {
//   privateDnsEnabled: true,
//   service: new ec2.InterfaceVpcEndpointService(`com.amazonaws.${region}.bedrock-runtime`, 443)
// });
// bedrockEndpoint.connections.allowDefaultPortFrom(ec2.Peer.ipv4(vpc.vpcCidrBlock), `allowBedrockPortFrom-${projectName}`)

// bedrockEndpoint.addToPolicy(
//   new iam.PolicyStatement({
//     principals: [new iam.AnyPrincipal()],
//     actions: ['bedrock:*'],
//     resources: ['*'],
//   }),
// ); 
```

아래 명령어로 endpoint을 설치합니다. 

```text
cdk deploy --all
```

이후 테스트 하여야 하니 이미 상기 조건으론 접속 가능하여 테스트가 안되었습니다. 안되는 테스트 환경을 만들어서 endpoint enable하고 다시 테스트 예정입니다.
