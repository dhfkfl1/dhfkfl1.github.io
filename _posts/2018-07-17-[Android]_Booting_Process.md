---
layout: post
title: "[180717] 안드로이드 프레임워크의 부팅 과정"
date:   2018-07-17 17:55:25 +0900
---

<h3>안드로이드의 부팅 과정</h3>

<br>
<img src="/assets/images/booting.JPG" width="950" height="650">
<br>

1. 리눅스 커널<br>
안드로이드는 리눅스 기반의 플랫폼으로, 부팅 시 부트로더를 통해 리눅스 커널이 먼저 시작된다.<br>
리눅스가 부팅되면 일반적인 리눅스 부팅 과정처럼 커널 초기화를 수행한 후 마지막 과정에서 init 프로세스를 호출한다.<br>

2. init<br>
안드로이드 init 프로세스는 각종 디바이스를 초기화하는 작업을 비롯해 안드로이드 프레임워크 동작에 필요한 각종 데몬, 컨텍스트 매니저, 미디어 서버, Zygote 등을 실행한다.<br>

3. 컨텍스트 매니저(Context Manager)<br>
컨텍스트 매니저는 안드로이드의 시스템 서비스를 관리하는 중요한 프로세스이다.<br>
시스템 서비스는 안드로이드 프레임워크를 구성하는 중요한 컴포넌트로서 카메라, 오디오, 비디오 처리에서부터 각종 애플리케이션 제작에 필요한 중요 API를 제공하는 등의 역할을 수행한다.<br>

4. 미디어 서버<br>
미디어 서버 프로세스는 안드로이드에서 오디오 출력이나 카메라 서비스와 같이 C/C++ 기반으로 작성되어 있는 네이티브 시스템 서비스를 실행하는 역할을 한다.<br>

5. Zygote<br>
Zygote는 안드로이드 애플리케이션의 로딩 시간을 단축하기 위한 프로세스로서 모든 자바 기반 안드로이드 애플리케이션은 Zygote를 통해 포크(fork)된 프로세스 상에서 동작한다.<br>

6. 시스템 서버(System Server)<br>
시스템 서버는 Zygote에서 최초로 포크되어 실행되는 안드로이드 애플리케이션 프로세스다.<br>
시스템 서버는 애플리케이션 생명 주기를 제어하는 액티비티 매니저 서비스나 단말기의 위치 정보를 제공하는 로케이션 매니저 서비스와 같은 자바 시스템 서비스를 실행하는 역할을 한다.<br>

<h3>인텐트 필터의 매칭 개선</h3>
인텐트의 정보와 애플리케이션의 인텐트 필터 영역에 등록된 정보를 매칭하여 인텐트의 전달이 일어나는데, 이를 개선하는 방법으로 비교하는 영역을 BitSet으로 만들어 이를 비교하면 성능개선을 할 수 있는 여지가 있을 것 같다는 의견엔 멘토님도 시도할 가치가 있다고 말씀하셨다. 다만, 애플리케이션에 대한 정보는 패키지 매니저(Package Manager)가 갖고 있고 이러한 정보의 Set이 어떤 형태로 저장되어 있는지 알기위해선 Package Manager.java를 살펴보아야 할 것 같다.<br>

~~~
public ComponentName resolveActivity(@NonNull PackageManager pm) {
    if (mComponent != null) {
        return mComponent;
    }

    ResolveInfo info = pm.resolveActivity(
        this, PackageManager.MATCH_DEFAULT_ONLY);
    if (info != null) {
        return new ComponentName(
                info.activityInfo.applicationInfo.packageName,
                info.activityInfo.name);
    }

    return null;
}
~~~
IntentFilter.java 내에선 패키지 매니저에 대한 코드는 찾아볼 수 없었으나, 위의 코드는 Intent.java의 일부분이다. 인텐트 클래스에서는 패키지 매니저로부터 정보를 가져오는 부분이 있는 것 같다.<br>
