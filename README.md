# sample_config
# ABC service

## migration - mapping

#### cloudwatch

| mgmt_org         | NEW                           |
| ---------------- | ----------------------------- |
| ./cloudwatch     | ABCM_cloudwatch              |

#### private_before_resource

| dev_org            | NEW                                   |
| ------------------ | ------------------------------------- |
| ./ecr/dev_ecr      | ABCD_private_before_ecr              |
| ./kms/dev          | ABCD_private_before_kms              |
| ./ecr/qa_ecr       | ABCQ_private_before_ecr              |
| ./kms/qa           | ABCQ_private_before_kms              |
| ./version_resource | ABCD_private_before_version_resource |

| mgmt_org           | NEW                                   |
| ------------------ | ------------------------------------- |
| ./ecr/mgmt_ecr     | ABCM_private_before_ecr              |
| ./version_resource | ABCM_private_before_version_resource |

| prd_org            | NEW                                   |
| ------------------ | ------------------------------------- |
| ./ecr              | ABCP_private_before_ecr              |
| ./kms              | ABCP_private_before_kms              |

- mgmt before에는 kms와 version_resource의 state가 없음
- prd before version_resource는 dev와 같음

#### public_after_resource

| dev_org               | NEW                                |
| --------------------- | ---------------------------------- |
| ./route53/host/public | ABCA_public_after_route53_public  |
| ./route53/host/private/dev | ABCD_public_after_route53_private |
| ./route53/host/private/qa | ABCQ_public_after_route53_private |
| ./cloudfront          | ABCA_public_after_cloudfront      |

| mgmt_org              | NEW                                |
| --------------------- | ---------------------------------- |
| ./route53/host/private | ABCM_public_after_route53_private |
| ./ap_northeast_2/region_service/tgw | ABCM_public_after_tgw(확인) |

| prd_org               | NEW                                |
| --------------------- | ---------------------------------- |
| ./route53/host/public | ABCP_public_after_route53_public  |
| ./route53/host/private| ABCP_public_after_route53_private |
| ./cloudfront          | ABCP_public_after_cloudfront      |

- mgmt route53의 public은 생성하지 않는것으로 보임
- tgw 권한 확인 필요

#### ap_northeast_2/region_service

| dev_org          | NEW                            |
| ---------------- | ------------------------------ |
| ./region_service | ABCD_apne2_region_service     |

| mgmt_org         | NEW                            |
| ---------------- | ------------------------------ |
| ./region_service | ABCM_apne2_region_service     |

| prd_org          | NEW                            |
| ---------------- | ------------------------------ |
| ./region_service | ABCP_apne2_region_service     |

- import mgmt_cgw cgw-053e2205c4dcd44b5 못찾음

#### ap_northeast_2/ABC_{dev,qa,mgmt,prd}_vpc/network

| org                         | NEW                                    |
| --------------------------- | -------------------------------------- |
| ./vpc.tf,eip,igw,nat,subnet | ABC{D,Q,M,P}_apne2_net_common         |
| ./vpn                       | ABC{D,Q,M,P}_apne2_net_vpn            |
| ./route_table               | ABC{D,Q,M,P}_apne2_net_route_table    |
| ./security_group            | ABC{D,Q,M,P}_apne2_net_security_group |
| ./endpoint                  | ABC{D,Q,M,P}_apne2_net_endpoint        |

- vpn은 state가 없음
- PRD의 route_table에서 다음의 형상이 없음
  - aws_route.private_db_rtb_tgw_route_dev
  - aws_route.private_db_rtb_tgw_route_mgmt
  - aws_route.private_rtb_tgw_route_dev
  - aws_route.private_rtb_tgw_route_mgmt
- PRD의 security_group에서 다음의 형상이 없음
  - aws_security_group_rule.fo_erp_common2_sg_rule_ingress_41
  - aws_security_group_rule.fo_erp_common2_sg_rule_ingress_45

#### ap_northeast_2/ABC_{dev,qa,mgmt,prd}_vpc/infra

| org              | NEW                                  |
| ---------------- | ------------------------------------ |
| ./eni            | ABC{D,Q,P}_apne2_inf_eni            |
| ./ec2            | ABC{D,Q,M,P}_apne2_inf_ec2            |
| ./loadbalance    | ABC{D,Q,M,P}_apne2_inf_lb             |
| ./efs            | ABC{D,Q,P}_apne2_inf_efs            |
| ./ecs            | ABC{D,Q,P}_apne2_inf_ecs            |

- eni는 mgmt에 리소스도 없고, 코드도 없음
- ABCM_apne2_inf_lb 중 일부 리소스가 코드 상에만 존재하고, 실제하지 않아 import 불가

#### all_region/iam

| dev_org            | NEW                                |
| ------------------ | ---------------------------------- |
| ./all_region/iam   | ABCD_all_region_iam               |


- RDS는 EC2 Instane로 대체, import 불필요

variable "workspace_db" {
  type = set(string)
  default = [
    "ABCD_db_oracle",          //
    "ABCD_db_aurora",          //
  ]
}



## Appendix

### Workspace rule

{서비스}{용도}\_{대분류}\_{중분류}\_{소분류}

- 서비스 : ABC

- 용도

  | 용도                     | 표기  |
  | ----------------------- | ---- |
  | 공통 (ALL)               | A    |
  | 개발 (Development)       | D    |
  | 검증 (Quality Assurance) | Q    |
  | 관리 (Management)        | M    |
  | 운영 (Production)        | P    |

- 대분류 :

  - Private
  - Public
  - ALL
  - {Region code}

-  중분류 :

  - 용도 : `before` `after`
  - 서비스 단위 : `net` `inf` `db`

- 소분류 :  리소스 이름



### region code

| Code  | Original Code    | 이름                            |
| :---- | :--------------- | :------------------------------ |
| usea2 | us-east-2      | 미국 동부(오하이오)             |
| usea1 | us-east-1      | 미국 동부(버지니아 북부)        |
| uswe1 | us-west-1      | 미국 서부(캘리포니아 북부 지역) |
| uswe2 | us-west-2      | 미국 서부(오레곤)               |
| afso1 | af-south-1     | 아프리카(케이프타운)            |
| apea1 | ap-east-1      | 아시아 태평양(홍콩)             |
| apso1 | ap-south-1     | 아시아 태평양(뭄바이)           |
| apne3 | ap-northeast-3 | 아시아 태평양(오사카-로컬)      |
| apne2 | ap-northeast-2 | 아시아 태평양(서울)             |
| apse1 | ap-southeast-1 | 아시아 태평양(싱가포르)         |
| apse2 | ap-southeast-2 | 아시아 태평양(시드니)           |
| apne1 | ap-northeast-1 | 아시아 태평양(도쿄)             |
| cace1 | ca-central-1   | 캐나다(중부)                    |
| euce1 | eu-central-1   | 유럽(프랑크푸르트)              |
| euwe1 | eu-west-1      | 유럽(아일랜드)                  |
| euwe2 | eu-west-2      | 유럽(런던)                      |
| euso1 | eu-south-1     | 유럽(밀라노)                    |
| euwe3 | eu-west-3      | 유럽(파리)                      |
| euno1 | eu-north-1     | 유럽(스톡홀름)                  |
| meso1 | me-south-1     | 중동(바레인)                    |
| saea1 | sa-east-1      | 남아메리카(상파울루)            |


# Azure Case

## Workspace rule

> {서비스|프로젝트명}{환경}\_{대분류}\_{중분류}\_{소분류}

- 서비스|프로젝트명 : poc

- 환경
 
|Environment |	Code|
| ----------------------- | ---- |
|Development|	dev / D|
|Test	|test / T|
| Quality Assurance | qa /Q   |
|System Integration Testing	|sit /I| 
|User Acceptance Test	|uat/ A|
|Staging |	stag /S|
|Production	|prod / P|
|Disaster Recovery	|dr /R|


- 대분류 : Region Code

<table class="tg">
  <tr>
    <th class="tg-fymr">Geography</th>
    <th class="tg-fymr">Country</th>
    <th class="tg-fymr">Region</th>
    <th class="tg-7btt">Code</th>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">Central US</td>
    <td class="tg-c3ow">CUS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">East US 2</td>
    <td class="tg-c3ow">EUS2</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">East US</td>
    <td class="tg-c3ow">EUS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">North Central US</td>
    <td class="tg-c3ow">NCUS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">South Central US</td>
    <td class="tg-c3ow">NSUS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">West US 2</td>
    <td class="tg-c3ow">WUS2</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">USA</td>
    <td class="tg-0pky">West Central US</td>
    <td class="tg-c3ow">WCUS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">West US,US DoD Central</td>
    <td class="tg-c3ow">WUSD</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US DoD East</td>
    <td class="tg-c3ow">USDE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US Gov. Arizona</td>
    <td class="tg-c3ow">USGA</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US Gov. Iowa</td>
    <td class="tg-c3ow">USGI</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US Gov. Texas</td>
    <td class="tg-c3ow">USGT</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US Gov. Virginia</td>
    <td class="tg-c3ow">USGV</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US Sec East</td>
    <td class="tg-c3ow">USSE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Azure Gov</td>
    <td class="tg-0pky">US Sec West</td>
    <td class="tg-c3ow">USSW</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Canada</td>
    <td class="tg-0pky">Canada Central</td>
    <td class="tg-c3ow">CC</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Canada</td>
    <td class="tg-0pky">Canade East</td>
    <td class="tg-c3ow">CE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Americas</td>
    <td class="tg-0pky">Brazil</td>
    <td class="tg-0pky">Brazil South</td>
    <td class="tg-c3ow">BS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">North Europe</td>
    <td class="tg-c3ow">NE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">West Europe</td>
    <td class="tg-c3ow">WE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">UK</td>
    <td class="tg-0pky">UK South</td>
    <td class="tg-c3ow">UKS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">UK</td>
    <td class="tg-0pky">UK West</td>
    <td class="tg-c3ow">UKW</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Germany</td>
    <td class="tg-0pky">Germany North</td>
    <td class="tg-c3ow">GN</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Germany</td>
    <td class="tg-0pky">Germany West Central</td>
    <td class="tg-c3ow">GWC</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Switzerland</td>
    <td class="tg-0pky">Switzerland North</td>
    <td class="tg-c3ow">SN</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Switzerland</td>
    <td class="tg-0pky">Switzerland West</td>
    <td class="tg-c3ow">SW</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Norway</td>
    <td class="tg-0pky">Norway West</td>
    <td class="tg-c3ow">NW</td>
  </tr>
  <tr>
    <td class="tg-0pky">Europe</td>
    <td class="tg-0pky">Norway</td>
    <td class="tg-0pky">Norway East</td>
    <td class="tg-c3ow">NE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">East Asia</td>
    <td class="tg-c3ow">EA</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Southeast Asia</td>
    <td class="tg-c3ow">SA</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Australia</td>
    <td class="tg-0pky">Australia Central</td>
    <td class="tg-c3ow">AC</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Australia</td>
    <td class="tg-0pky">Australia Central 2</td>
    <td class="tg-c3ow">AC2</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Australia</td>
    <td class="tg-0pky">Australia East</td>
    <td class="tg-c3ow">AE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Australia</td>
    <td class="tg-0pky">Australia Southeast</td>
    <td class="tg-c3ow">AS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">China</td>
    <td class="tg-0pky">China East</td>
    <td class="tg-c3ow">CE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">China</td>
    <td class="tg-0pky">China North</td>
    <td class="tg-c3ow">CN</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">China</td>
    <td class="tg-0pky">China East 2</td>
    <td class="tg-c3ow">CE2</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">China</td>
    <td class="tg-0pky">China North 2</td>
    <td class="tg-c3ow">CN2</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">India</td>
    <td class="tg-0pky">Central India</td>
    <td class="tg-c3ow">CI</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">India</td>
    <td class="tg-0pky">South India</td>
    <td class="tg-c3ow">SI</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">India</td>
    <td class="tg-0pky">West India</td>
    <td class="tg-c3ow">WI</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Japan</td>
    <td class="tg-0pky">Japan East</td>
    <td class="tg-c3ow">JE</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Japan</td>
    <td class="tg-0pky">Japan West</td>
    <td class="tg-c3ow">JW</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Korea</td>
    <td class="tg-0pky">Korea Central</td>
    <td class="tg-c3ow">KC</td>
  </tr>
  <tr>
    <td class="tg-0pky">Asia Pacific</td>
    <td class="tg-0pky">Korea</td>
    <td class="tg-0pky">Korea South</td>
    <td class="tg-c3ow">KS</td>
  </tr>
  <tr>
    <td class="tg-0pky">Middle East and Africa</td>
    <td class="tg-0pky">South Africa</td>
    <td class="tg-0pky">South Africa North</td>
    <td class="tg-c3ow">SAN</td>
  </tr>
  <tr>
    <td class="tg-0pky">Middle East and Africa</td>
    <td class="tg-0pky">South Africa</td>
    <td class="tg-0pky">South Africa West</td>
    <td class="tg-c3ow">SAW</td>
  </tr>
  <tr>
    <td class="tg-0pky">Middle East and Africa</td>
    <td class="tg-0pky">UAE</td>
    <td class="tg-0pky">UAE Central</td>
    <td class="tg-c3ow">UC</td>
  </tr>
  <tr>
    <td class="tg-0pky">Middle East and Africa</td>
    <td class="tg-0pky">UAE</td>
    <td class="tg-0pky">UAE North</td>
    <td class="tg-c3ow">UN</td>
  </tr>
</table>

- 중분류 : 용도
   - Private (pri)
   - Public (pub)
   - common (공통) com
-  소분류 : 
   - 서비스 단위 `net` `inf` `db`
   - 리소스 타입: `aks` `vm` `db`


```
예) 
pocD_KC_com_inf_aks : PoC 개발 환경, 한국 중부,공통, 인프라, AKS
pocD_KS_pri_net_vm: PoC 개발 환경, 한국 남부, 프라이빗, 네트워크 인프라, VM 리소스
```





---

### Reference
- Azure [Define your naming convention](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming) 
<img src="https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/_images/ready/resource-naming.png">

- Azure [Naming rules and restrictions for Azure resources](https://docs.microsoft.com/en-gb/azure/azure-resource-manager/management/resource-name-rules)
- AWS [Standardize Names for AWS Resources for EC2 Instance](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/ec2-instances.html)
<img src="https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/images/tagimage3.png">

- AWS [Standardize Names for AWS Resources for Other AWS Resources](https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/other-aws-resource-types.html) : account-name(enviroment) : resource-name	: resource-type
- [Cloud Naming Convention](https://stepan.wtf/cloud-naming-convention/) : [prefix]-[project]-[env]-[resource]-[location]-[description]-[suffix]
- [Workspace Naming](https://www.terraform.io/docs/cloud/workspaces/naming.html) : COMPONENT-ENVIRONMENT-REGION


### AKS Related
 - [Azure의 Terraform 설명서](https://docs.microsoft.com/ko-kr/azure/developer/terraform/)
 - [terraform-providers/terraform-provider-azurerm](https://github.com/terraform-providers/terraform-provider-azurerm/tree/master/examples/kubernetes)
 - [Manage Kubernetes Resources via Terraform](https://learn.hashicorp.com/tutorials/terraform/kubernetes-provider?in=terraform/kubernetes)
 - [Provision an AKS Cluster (Azure)](https://learn.hashicorp.com/tutorials/terraform/aks?in=terraform/kubernetes)



### Azure Region Code
```
DisplayName               Name                 RegionalDisplayName
------------------------  -------------------  -------------------------------------
East US                   eastus               (US) East US
East US 2                 eastus2              (US) East US 2
East US STG               eastusstg            (US) East US STG
South Central US          southcentralus       (US) South Central US
South Central US STG      southcentralusstg    (US) South Central US STG
West US 2                 westus2              (US) West US 2
Australia East            australiaeast        (Asia Pacific) Australia East
Southeast Asia            southeastasia        (Asia Pacific) Southeast Asia
North Europe              northeurope          (Europe) North Europe
UK South                  uksouth              (Europe) UK South
West Europe               westeurope           (Europe) West Europe
Central US                centralus            (US) Central US
North Central US          northcentralus       (US) North Central US
West US                   westus               (US) West US
South Africa North        southafricanorth     (Africa) South Africa North
Central India             centralindia         (Asia Pacific) Central India
East Asia                 eastasia             (Asia Pacific) East Asia
Japan East                japaneast            (Asia Pacific) Japan East
Korea Central             koreacentral         (Asia Pacific) Korea Central
Canada Central            canadacentral        (Canada) Canada Central
France Central            francecentral        (Europe) France Central
Germany West Central      germanywestcentral   (Europe) Germany West Central
Norway East               norwayeast           (Europe) Norway East
Switzerland North         switzerlandnorth     (Europe) Switzerland North
UAE North                 uaenorth             (Middle East) UAE North
Brazil South              brazilsouth          (South America) Brazil South
Central US (Stage)        centralusstage       (US) Central US (Stage)
East US (Stage)           eastusstage          (US) East US (Stage)
East US 2 (Stage)         eastus2stage         (US) East US 2 (Stage)
North Central US (Stage)  northcentralusstage  (US) North Central US (Stage)
South Central US (Stage)  southcentralusstage  (US) South Central US (Stage)
West US (Stage)           westusstage          (US) West US (Stage)
West US 2 (Stage)         westus2stage         (US) West US 2 (Stage)
Asia                      asia                 Asia
Asia Pacific              asiapacific          Asia Pacific
Australia                 australia            Australia
Brazil                    brazil               Brazil
Canada                    canada               Canada
Europe                    europe               Europe
Global                    global               Global
India                     india                India
Japan                     japan                Japan
United Kingdom            uk                   United Kingdom
United States             unitedstates         United States
East Asia (Stage)         eastasiastage        (Asia Pacific) East Asia (Stage)
Southeast Asia (Stage)    southeastasiastage   (Asia Pacific) Southeast Asia (Stage)
Central US EUAP           centraluseuap        (US) Central US EUAP
East US 2 EUAP            eastus2euap          (US) East US 2 EUAP
West Central US           westcentralus        (US) West Central US
South Africa West         southafricawest      (Africa) South Africa West
Australia Central         australiacentral     (Asia Pacific) Australia Central
Australia Central 2       australiacentral2    (Asia Pacific) Australia Central 2
Australia Southeast       australiasoutheast   (Asia Pacific) Australia Southeast
Japan West                japanwest            (Asia Pacific) Japan West
Korea South               koreasouth           (Asia Pacific) Korea South
South India               southindia           (Asia Pacific) South India
West India                westindia            (Asia Pacific) West India
Canada East               canadaeast           (Canada) Canada East
France South              francesouth          (Europe) France South
Germany North             germanynorth         (Europe) Germany North
Norway West               norwaywest           (Europe) Norway West
Switzerland West          switzerlandwest      (Europe) Switzerland West
UK West                   ukwest               (Europe) UK West
UAE Central               uaecentral           (Middle East) UAE Central
Brazil Southeast          brazilsoutheast      (South America) Brazil Southeast
```

```
code validation sample
variable "common_tags" {
  description = "필수 공통 TAG 7개"
  type        = map(any)
}

variable "prefix" {
  description = "업무 구분 mis/cmn"
}

variable "env" {
  description = "배포 환경에 대한 설정. prod/dev/poc"
}

variable "loc_code" {
  description = "배포 지역에 대한 코드 중부 (koce)/남부 (koso)"
  
  validation {
    condition = var.loc_code == "koce" || var.loc_code == "koso"
    error_message = "Error : \n 지원되지 않는 배포 지역입니다. \n 다음 배포 지역(Region)만 사용 가능합니다. \n \t - 한국 중부 : koce \n \t - 한국 남부 : koso \n ."
  }
}

variable "location" {
  description = "배포 지역 설정 한국 중부(koreacentral), 한국 남부(koreasouth)"
  
  validation {
    condition = contains(["koreacentral", "Korea Central", "koreasouth", "Korea South"], var.location)
    error_message = "Error : \n 지원되지 않는 배포 지역입니다. \n 다음 배포 지역(Region)만 사용 가능합니다. \n \t - 한국 중부 : koreacentral , Korea Central \n \t - 한국 남부 : koreasouth, Korea South \n ."
  }
}

# misO_KC_inf_aks용 Varaibles
variable "k8s_version" {
  description = "AKS에서 사용할 K8S버전 지정"
  validation {
    condition = contains(["17","18","19"], substr(var.k8s_version,2,2))
    error_message = "Error : \n 지원되지 않는 Kubernetes 버전입니다. \n 다음 버전만 사용 가능합니다. \n \t - 1.18.10 \n \t - 1.19.1 \n \t - 1.19.3 \n ."
  }
}

variable "aks_vm_size" {
  description = "default_node_pool"
  type = string
  
  # Validation error message must be at least one full English sentence starting with an uppercase letter and ending with a period or question mark.(\".)
  validation {
    condition = contains(["Standard_D4s_v3" , "Standard_D2s_v3" , "Standard_D3_v2", "Standard_D16s_v3", "Standard_E8s_v3", "Standard_F4s"], var.aks_vm_size)
    error_message = "Error : \n 지원되지 않는 VM Size입니다. \n 다음 VM Size만 사용 가능합니다. \n \t - Standard_D4s_v3 \n \t - Standard_D2s_v3 \n \t - Standard_D3_v2\n \t - Standard_D16s_v3\n \t - Standard_E8s_v3 \n \t - Standard_F4s \n ."
  }
}

variable "fwprivate_ip" {
  description = "Firewall private ip for AKS Node Pool" 
}
```


https://docs.microsoft.com/ko-kr/azure/app-service/scripts/terraform-secure-backend-frontend

```
output "ws_names" {
  #value = azurerm_container_registry.mis_poc_acr.id
   value = [ for ws_names in setunion(var.workspace_common, var.workspace_net, var.workspace_inf) : ws_names]
}

```

