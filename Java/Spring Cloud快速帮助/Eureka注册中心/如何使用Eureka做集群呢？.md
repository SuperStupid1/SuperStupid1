## 1�����ʹ��Eureka����Ⱥ�أ�

	����ֻ��Ҫ��������Eureka����Ȼ������������ķ���ע���ʱ���໥ע��Ϳ����ˣ�����������������һ��Eureka

server:
  port: 8177
spring:
  application:
    name: Eureka-Register

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://127.0.0.1:8176/eureka/


	�������ǿ��Կ������Ǵ�����һ��Eureka����������ע��Ķ˿ڲ������Լ���
	
	��ô�����ٴ���һ����Ⱥ��ע������


server:
  port: 8176
spring:
  application:
    name: Eureka-Register-colony1

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://127.0.0.1:8177/eureka/



	�������Ǿͺ�����ľ��ܿ���������ʹ����8177����˿ڵ�Eurekaע������Ȼ������ע�ᵽ8176����
	
	Ȼ�������ִ�����һ��Eurekaʹ����8176Ȼ��ע�ᵽ��8177�����������ǵ�ע�����ľ������˼�Ⱥ��



	Ȼ������ȥ������������ȥ����������ע��ĵ�ַ


eureka:
  client:
    service-url:
      defaultZone:  http://127.0.0.1:8177/eureka/,ttp://127.0.0.1:8176/eureka/



	����������ܺ�����Ŀ��õ�����ע��������ע�����ģ��������Ǿ�����һ���򵥵���ؤ��Eureka��Ⱥ��



























