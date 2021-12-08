# 2021-12-08 (ARM v8-M TrustZone)

ARM v8-A 는 Exception Level 1, 2, 3 으로 나뉘고, Secure world와 Non Secure world로 나뉜다.

그러나 ARM v8-M 은 Exception Level로 구분하지 않는다.

> There are three security attributes a memory region can have. One for each security level (secure and non-secure) and an additional (non-secure callable), which we are going to discuss soon:
>
> - non-secure (**NS**)
> - non-secure callable (**NSC**)
> - secure (**S**)

non-secure world 에서 secure world 로 가려면 Call 을 호출해야한다. ARM v8-A 에서 Hypervisor Call (HVC), Secure monitor Call (SMC) 등과 같이 ARM v8-M 은 SG (Secure Gateway) 를 호출한다.

> ```
> secure_function:
>     SG
>     B.W __acle_se_secure_function
> ```

https://embeddedsecurity.io/sec-tz-basics.html