### 13.9.1IBM WebSphere

On WebSphere 6.1.0.9 and above, the recommended Spring JTA transaction manager to use is`WebSphereUowTransactionManager`. This special adapter leverages IBM’s`UOWManager`API, which is available in WebSphere Application Server 6.1.0.9 and later. With this adapter, Spring-driven transaction suspension \(suspend/resume as initiated by`PROPAGATION_REQUIRES_NEW`\) is officially supported by IBM.

