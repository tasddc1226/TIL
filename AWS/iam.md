# AWS IAM
> IAM = Identity and Access Management, Global Service

## IAM의 사용 가능한 구성
- IAM에는 한 사용자는 하나의 조직(Group)에만 속할 수 있다.
- **한 그룹은 다른 그룹을 포함시킬 수 없다. 단지 사용자만 포함할 수 있다.**
- 어떤 사용자가 특정 그룹에 포함되어있지 않는 방법도 가능하지만 추천하는 방법은 아니다.
- **또한 한 사용자는 다른 여러 그룹에 속할 수 있다.**

### 사용자와 그룹을 생성하는 이유?
- AWS 계정을 사용하도록 허용하기 위함이다.
- 그리고 허용을 위해서 이들에게 권한을 부여해야 한다.
- 부여하는 방법으로는 JSON 문서를 통해서 가능하다.

```json
{
    "Version": "2017-10-10",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:Describe*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "elasticloadbalancing:Describe*",
            "Resource": "*"
        }
    ]
}
```

- 어느 특정 유저에게 권한을 너무 많이 설정한다면 큰 비용 또는 보안 문제를 야기할 수 있기 때문에
- AWS는 **최소 권한의 원칙**을 적용해야 한다.
- 사용자가 꼭 필요로 하는 것 이상의 권한을 주지 않도록 해야한다.