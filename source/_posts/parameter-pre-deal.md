title: 通过ModelAttribute对请求参数进行预处理
date: 2015-08-10 17:42:42
tags: [Spring MVC]
---
{% codeblock UserController.java %}
package org.springside.examples.showcase.web;

import java.util.List;
......

import com.google.common.collect.Maps;

@Controller
@RequestMapping(value = "/account/user")
public class UserController {
	
	......
	
	@RequestMapping(value = "update", method = RequestMethod.POST)
	public String update(@Valid @ModelAttribute("user") User user,
			@RequestParam(value = "roleList") List<Long> checkedRoleList, RedirectAttributes redirectAttributes) {

		// bind roleList
		user.getRoleList().clear();
		for (Long roleId : checkedRoleList) {
			Role role = new Role(roleId);
			user.getRoleList().add(role);
		}

		accountService.saveUser(user);

		redirectAttributes.addFlashAttribute("message", "保存用户成功");
		return "redirect:/account/user";
	}

	/**
	 * 所有RequestMapping方法调用前的Model准备方法, 实现Struts2 Preparable二次部分绑定的效果,先根据form的id从数据库查出User对象,再把Form提交的内容绑定到该对象上。
	 * 因为仅update()方法的form中有id属性，因此仅在update时实际执行.
	 */
	@ModelAttribute
	public void getUser(@RequestParam(value = "id", defaultValue = "-1") Long id, Model model) {
		if (id != -1) {
			model.addAttribute("user", accountService.getUser(id));
		}
	}

	/**
	 * User类中有roleList,不自动绑定对象中的roleList属性，另行处理。
	 */
	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.setDisallowedFields("roleList");
	}
}
{% endcodeblock %}

其中getUser方法可写成
{% codeblock lang:java %}
	@ModelAttribute
	public User getUser(@RequestParam(value = "id", defaultValue = "-1") Long id, Model model) {
		if (id != -1) 
			return accountService.getUser(id);
	}
{% endcodeblock %}

这种情况，函数隐含Model类型的参数，会调用addAttribute方法，addAttribute的key没有指定，它由返回类型隐含表示，如这个方法返回User类型，那么这个key是user，value为返回对象，其中key值也可以通过@ModelAttribute注解的value参数指定（eg:@ModelAttribute("user")）。
<br><br>
update方法如果没有标注@SessionAttributes("user")，那么scope为request，如果标注了，那么scope为session。
