<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<!-- Render JSP file -->
		<dependency>
			<groupId>org.apache.tomcat.embed</groupId>
			<artifactId>tomcat-embed-jasper</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>jakarta.servlet.jsp.jstl</groupId>
			<artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
		</dependency>
		<dependency>
			<groupId>org.glassfish.web</groupId>
			<artifactId>jakarta.servlet.jsp.jstl</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mariadb.jdbc</groupId>
			<artifactId>mariadb-java-client</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>


==================================================


 spring.application.name=SpringWebMVCDemo

# Setting mariaDB
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3307/employees
spring.datasource.username=root
spring.datasource.password=sapassword
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true

#And, Spring Boot, provides defaults for both:
# - spring.jpa.hibernate.naming.physical-strategy defaults to org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy, and
# - spring.jpa.hibernate.naming.implicit-strategy defaults to org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
#
#We can override these values, but by default, these will:
# - Replace dots with underscores
# - Change camel case to snake case, and
# - Lower-case table names


# Spring MVC
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp



=====================================================


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;

import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;

import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class SecurityConfig {

	private static final String ROLE_ADMIN = "ADMIN";
	private static final String ROLE_USER = "USER";

	@Autowired
	public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
		UserDetails userDetails01 = User
				.builder()
				.username("admin")
				.password("$2a$12$f2N6Rp5icyQZq6u/YuJUM.ATiTGz/j8pMOw4KzJWP1/uUL4b23xju")
				.roles(ROLE_ADMIN)
				.build();

		UserDetails userDetails02 = User.builder()
				.username("user01")
				.password("$2a$12$f2N6Rp5icyQZq6u/YuJUM.ATiTGz/j8pMOw4KzJWP1/uUL4b23xju")
				.roles(ROLE_USER)
				.build();

		auth.inMemoryAuthentication()
		.withUser(userDetails01)
		.withUser(userDetails02);
	}

	@Bean
	SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
		httpSecurity.csrf(csrf -> {
			csrf.disable();
		})
		// Require authentication
		.authorizeHttpRequests(request -> request
				.requestMatchers("/employees/**").hasRole(ROLE_ADMIN)
				.requestMatchers("/home-role-user").hasAnyRole(ROLE_ADMIN, ROLE_USER)
				.anyRequest()
				.permitAll()
		)	
		// Specify the login page and permit all access to it
		.formLogin(form -> {
					form.loginProcessingUrl("/login");
					form.defaultSuccessUrl("/employees");
					form.permitAll();
				})
		// Specify the logout request matcher and permit all access to it
		.logout(logout -> logout.permitAll())
		.httpBasic(Customizer.withDefaults());

		return httpSecurity.build();
	}

	@Bean
	PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
}

=====================================================




import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import iuh.fit.se.entities.Employee;
import iuh.fit.se.services.EmployeeService;
import jakarta.validation.Valid;

@Controller
@RequestMapping("/employees")
public class EmployeeController {

	@Autowired
	EmployeeService employeeService;

	@GetMapping
	public ModelAndView getList(ModelAndView model) {
		List<Employee> employees = employeeService.findAll();
		model.setViewName("employee-list");
		model.addObject("employees", employees);
		return model;
	}
	
	@GetMapping("/page/{pageNo}")
	public String getList(Model model, 
			@PathVariable int pageNo,
		    @RequestParam(defaultValue = "2", required = false) int pageSize,
            @RequestParam(defaultValue = "id", required = false) String sortBy,
            @RequestParam(defaultValue = "ASC", required = false) String sortDirection) {
		Page<Employee> employees = employeeService.findAllWithPaging(pageNo - 1, pageSize, sortBy, sortDirection);
		System.out.println(employees);
		
		model.addAttribute("currentPage", pageNo);
        
		model.addAttribute("totalPages", employees.getTotalPages());
        model.addAttribute("totalItems", employees.getTotalElements());

        model.addAttribute("sortBy", sortBy);
        model.addAttribute("sortDirection", sortDirection);
        
		model.addAttribute("employees", employees.getContent());
		
		return "employee-list";
	}
	

//	@RequestMapping(value = "/showForm", method = RequestMethod.GET)
	@GetMapping("/showForm")
	public ModelAndView showForm(ModelAndView model) {
		Employee employee = new Employee();
		model.setViewName("employee-form");
		model.addObject("employee", employee);
		return model;
	}

	@PostMapping("/save")
	public String save(@Valid @ModelAttribute Employee employee, BindingResult result) {
		if (result.hasErrors()) {
			return "employee-form";
		}
		
		if(employee.getAddress().getAddress().isEmpty()) {
			employee.setAddress(null);
		}
		
		employeeService.save(employee);
		
		return "redirect:/employees";
	}
	
	@GetMapping("/update")
	public ModelAndView update(@RequestParam("employeeId") int id, ModelAndView model) {
		Employee employee = employeeService.findById(id);
	
		if (employee == null) {
			model.addObject("message", "Can not find Employee with id: " + id);
			model.setViewName("error");
		}
		else {
			model.addObject("employee", employee);
			model.setViewName("employee-form");
		}
		
		return model;
	}

	@GetMapping("/delete")
	public String delete(@RequestParam("employeeId") int id) {
		employeeService.delete(id);
		return "redirect:/employees";
	}
	
	@GetMapping("/search")
	public ModelAndView search(@RequestParam String keyword, ModelAndView model) {
		List<Employee> employees = employeeService.search(keyword);
	    model.setViewName("employee-list");
		model.addObject("employees", employees);
	 
	    return model;    
	}
}
