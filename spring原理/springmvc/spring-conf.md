
#springmvc 常用的配置

* 资源配置
      <context:property-placeholder
            location="classpath:/resources/application.properties,classpath:/resources/redis.properties"/>
      <context:annotation-config/>

* aop扫描
      <context:component-scan base-package="com.mall.web.pc" use-default-filters="false">
          <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
      </context:component-scan>

* 上传文件的大小设置
      <!-- SpringMVC在超出上传文件限制时，会抛出org.springframework.web.multipart.MaxUploadSizeExceededException -->  
      <!-- 该异常是SpringMVC在检查上传的文件信息时抛出来的，而且此时还没有进入到Controller方法中 -->  
      <bean id="exceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">  
          <property name="exceptionMappings">  
              <props>  
                  <!-- 遇到MaxUploadSizeExceededException异常时，自动跳转到/error_fileupload.jsp页面 -->  
                  <prop key="org.springframework.web.multipart.MaxUploadSizeExceededException">error_fileupload</prop>
                  <prop key="com.mall.support.core.exceptions.BusinessException">error/500</prop>
                  <!--页面不存在-->
                  <prop key="com.mall.web.pc.pipe.NotFindViewException">error/404</prop>
                  <prop key="java.lang.Exception">error/500</prop>
              </props>
          </property>  
      </bean>

* 拦截器
      <mvc:interceptors>
          <mvc:interceptor>
              <mvc:mapping path="/order.shtml" />
              <mvc:mapping path="/addOrder.shtml" />
              <mvc:mapping path="/addCar/ajax.shtml" />
              <mvc:mapping path="/collectPro/ajax" />
              <mvc:mapping path="/car.shtml" />
              <mvc:mapping path="/member/addressSave/ajax" />
              <mvc:mapping path="/ajax/getAddress"/>
              <mvc:mapping path="/ajax/delAddress"/>
              <mvc:mapping path="/ajax/addShopFav.shtml" />
              <mvc:mapping path="/ajax/checkReceiveCouponCard.shtml" />
              <mvc:mapping path="/ajax/addContent.shtml" />
              <bean class="com.mall.web.pc.interceptor.SystemInterceptor" />
          </mvc:interceptor>
      </mvc:interceptors>

* 试图解析器
      <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="order" value="3"/>
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
      </bean>

* 静态资源寻址
      <mvc:resources mapping="/themes/default/css/**" location="/themes/default/css/" />
      <mvc:resources mapping="/themes/default/img/**" location="/themes/default/img/" />
      <mvc:resources mapping="/static/js/**" location="/static/js/" />
