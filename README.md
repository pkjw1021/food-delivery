

# 난리난 쇼밍몰_개인과제  (서비스 추가)


![image](https://user-images.githubusercontent.com/52017160/109935383-e2072580-7d10-11eb-9da0-0a7f6c1f3877.png)

**1) DDD 의 적용** 
```
package crazyshoppingmall;

import javax.persistence.*;
import org.springframework.beans.BeanUtils;
import java.util.List;
import java.util.Date;

@Entity
@Table(name="RiderMgmt_table")
public class RiderMgmt {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String orderId;
    private String addr;
    private Date startDt;
    private Date endDt;
    private String status;

    @PrePersist
    public void onPrePersist(){
        Rode rode = new Rode();
        BeanUtils.copyProperties(this, rode);
        rode.publishAfterCommit();


        CouponCreated couponCreated = new CouponCreated();
        BeanUtils.copyProperties(this, couponCreated);
        couponCreated.publishAfterCommit();


    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }
    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }
    public String getAddr() {
        return addr;
    }

    public void setAddr(String addr) {
        this.addr = addr;
    }
    public Date getStartDt() {
        return startDt;
    }

    public void setStartDt(Date startDt) {
        this.startDt = startDt;
    }
    public Date getEndDt() {
        return endDt;
    }

    public void setEndDt(Date endDt) {
        this.endDt = endDt;
    }
    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }
}


```
- Entity Pattern 과 Repository Pattern 을 적용하여 JPA 를 통하여 다양한 데이터소스 유형 (RDB or NoSQL) 에 대한 별도의 처리가 없도록 데이터 접근 어댑터를 자동 생성하기 위하여 Spring Data REST 의 RestRepository 를 적용하였다
```
package crazyshoppingmall;

import org.springframework.data.repository.PagingAndSortingRepository;

public interface RiderMgmtRepository extends PagingAndSortingRepository<RiderMgmt, Long>{

}
```
- 적용 후 REST API 의 테스트

#rider 서비스 처리, riderpage 서비스 처리
![image](https://user-images.githubusercontent.com/52017160/109934240-a324a000-7d0f-11eb-9501-725250dcc4cd.png)

## 2) CQRS
##   2.1) rider들의 정보를 볼수 있는 조회용 페이지 riderpage서비스 개발 
![image](https://user-images.githubusercontent.com/52017160/110056550-635ac880-7da2-11eb-9856-c52e7be42d18.png)

## 3) Correlation
## MSA 서비스 구현하면 했다고 보는 항목


## 4)REQ/RESP (동기 호출)
##   4.1) Delivery서비스의 배달 시작 됐을 시(deliveryStarted) rider서비스의 deliveryaccept 호출 하여 처리 한다.
##        Controller에 구현하여 처리 함
##   4.2) delivery 서비스에 동기 호출  
  ![image](https://user-images.githubusercontent.com/52017160/110071528-8c3d8680-7dbf-11eb-9f4f-b7d3dc5f2a23.png)  
##   4.3) delivery 서비스에 external 구현
  ![image](https://user-images.githubusercontent.com/52017160/110071482-78922000-7dbf-11eb-86fb-7128b5ae0362.png)
##   4.4 rider 서비스의 controller 구현
  ![image](https://user-images.githubusercontent.com/52017160/110071627-b5f6ad80-7dbf-11eb-9a23-4b8b7d7cdc46.png)
##   4.5) 정상적으로 작동 되는 거 확인
  ![image](https://user-images.githubusercontent.com/52017160/110056730-ad43ae80-7da2-11eb-9294-20fcdd639944.png)


## 5)GATEWAY
##   5.1) GATEWAY 생성 확인
   ![image](https://user-images.githubusercontent.com/52017160/109967213-7e8eef00-7d34-11eb-81c4-16ba4d524c4f.png)
    
##   5.2) GATEWAY를 통한 riderMgmts, deliveryViews 서비스 확인
   ![image](https://user-images.githubusercontent.com/52017160/109967862-4a67fe00-7d35-11eb-8c0d-f69a479243e2.png)

## 6) DEPLOY  과정 설명
##  -- Azure 컨테이너 레지스트리 로그인
##   az acr login --name skuser05
##   -- PAY build
##   docker build -t skuser05.azurecr.io/pay:v1 .
##   -- pay push
##   docker push skuser05.azurecr.io/pay:v1 
##   -- pay deployment 생성
##   kubectl create deploy pay --image=skuser05.azurecr.io/pay:v1
##   -- pay service 실행
##   kubectl expose deploy pay --type=ClusterIP --port=8080
##   -- pay service 확인
##   kubectl get all
##   -- 결과물 확인
  ![image](https://user-images.githubusercontent.com/52017160/109954049-2ea82c00-7d24-11eb-9e22-5e1c0c0f9808.png)

## 7) circuit breaker
##   7.1) delivery서비스 applicationl.yml 파일에 circuit breaker 설정 값 추가
   ![image](https://user-images.githubusercontent.com/52017160/110071412-513b5300-7dbf-11eb-9698-a54000756ddc.png)
##   7.2) rider서비스 부하 테스트를 위한 delay time 추가
   ![image](https://user-images.githubusercontent.com/52017160/110071291-1b966a00-7dbf-11eb-94ff-6063365e56d3.png)
##   7.3) delivery서비스 siege로 부하를 준 후 결과 확인- 서비스가 죽지 않고 circuit breaker가 작동하는 것을 확인 할 수 있다.
   ![image](https://user-images.githubusercontent.com/52017160/110071148-e558ea80-7dbe-11eb-8403-5ac57b98542e.png)
   
## 8) Autoscale (HPA)
##   8.1) 부하를 걸기 위해 리소스 줄여서 재 배포 (rider 서비스)  
  ![image](https://user-images.githubusercontent.com/52017160/110058489-be41ef00-7da5-11eb-8430-3756e1234585.png)
##   8.2) rider 시스템에 replica를 자동으로 늘려줄 수 있도록 HPA를 설정( CPU 사용량이 15%를 넘을 경우 replica를 10개까지 늘려 줌)
  ![image](https://user-images.githubusercontent.com/52017160/110060285-06164580-7da9-11eb-803f-8883a756bdb3.png)
##   8.3) 부하 가중 중
  ![image](https://user-images.githubusercontent.com/52017160/110060398-33fb8a00-7da9-11eb-87b8-b288226bd64b.png)
##   8.4) Autoscale 확인  
##        Autoscale되어 target에는 200% 증가 했으며, pod가 여러개로 증가한 모습을 볼 수 있다.
  ![image](https://user-images.githubusercontent.com/52017160/110060489-59889380-7da9-11eb-984d-d12dff08cf1f.png)



## 9) Zero-downtime deploy (readiness probe)
##   9.1) order 서비스의 deployment.yml 파일에 설정 한다
  ![image](https://user-images.githubusercontent.com/52017160/109991991-2bc23100-7d4e-11eb-9ab2-663dd015c153.png)
  
##   9.2) order 서비스 재 배포 시 pod가 2개 뜨고, 새로 띄운 pod가 준비 될 동안 기존 pod 유지
  root@labs-120884432:/home/project/personal/order/kubernetes# kubectl apply -f deployment.yml
  deployment.apps/order configured
  ![image](https://user-images.githubusercontent.com/52017160/109992538-b60a9500-7d4e-11eb-9709-9f924b806fe0.png)
##   9.3) 재 배포한 order 서비스가 정상 동작 확인
  ![image](https://user-images.githubusercontent.com/52017160/109992761-ed794180-7d4e-11eb-96f4-997f194be9c0.png)


## 10) ConfigMap 적용
##   10.1) delivery 서비스의 application.yaml에 ConfigMap 적용 대상 항목 추가
  ![image](https://user-images.githubusercontent.com/52017160/110050456-3ce36000-7d97-11eb-9aa9-007cd0a1d591.png)
##   10.2) delivery 서비스의 deployment.yaml에 ConfigMap 적용 대상 항목 추가
  ![image](https://user-images.githubusercontent.com/52017160/110050537-71571c00-7d97-11eb-85e7-b536c5ddf17c.png)
##   10.3) ConfigMap 생성  
  ![image](https://user-images.githubusercontent.com/52017160/110050715-d743a380-7d97-11eb-917c-13ccee332929.png)
##   10.4) ConfigMap 생성 확인
  ![image](https://user-images.githubusercontent.com/52017160/110050752-ec203700-7d97-11eb-80df-c82bdbc64c97.png)
##   10.5) 소스 안에 ConfigMap 사용 하여 구동함
  ![image](https://user-images.githubusercontent.com/52017160/110050808-0823d880-7d98-11eb-98d0-f61642ea48bd.png)


## 11) 폴리글랏 퍼시스턴스
## riderpage서비스에 기존의 h2 db 에서 hsqldb로 변경하여 사용
## pom.xml파일에 dependency를 변경 하였다.
![image](https://user-images.githubusercontent.com/52017160/109934820-45448800-7d10-11eb-91fc-b47506397e88.png)

## 12) self-healing (liveness probe)
##    12.1) rider서비스의 deployment.yml 파일에 liveness probe 설정을 8090 바꿈
![image](https://user-images.githubusercontent.com/52017160/109993624-d850e280-7d4f-11eb-9a98-b6618feb5a8b.png)
##    12.2) rider서비스에 liveness가 적용된 것을 확인
![image](https://user-images.githubusercontent.com/52017160/109995150-611c4e00-7d51-11eb-9a28-da91a291fdf6.png)
##    12.3) rider서비스에 liveness가 발동되었고 Restart가 발생함
![image](https://user-images.githubusercontent.com/52017160/109995410-9aed5480-7d51-11eb-8636-9f05f572a3ee.png)


