###### made by 곽주형
# DeskApp

## 개요 및 주요기능
### 개요
사내에서 사용하는 업무메신저를 확장하여 각각의 프로젝트에 인원을 추가할수 있고 진행상황과 서로의 스케줄을 공유하기 위해 만들었습니다.
### 주요기능
+ 로그인 및 회원가입
+ 실시간 접속여부 확인
+ 1대1 채팅과 알림 전송
+ 프로젝트 추가,수정,열람 권한부여
+ 근태 리스트 및 일정
## 기술스택

 ![기술스택](https://user-images.githubusercontent.com/72774535/108622363-a8662d00-747b-11eb-9a7d-59788d91204c.png)

#### 그 외 API
| 라이브러리 | 버전 |
| ------ | ------ |
| jstl | 1.2 |
| JSON-simple | 16.11 |
| JdatePicker | 5.1 |
| Spring WebSocket | 3.3 |
| Lombok Maven Plugin | 4.4 |
| Mybatis Spring | 3.6 |
| ojdbc6 | 4.1.5 |
| Apache commons Fileupload | 4.1.5 |
| Naver Smart editer | 2.2 |

### Member
#### 곽주형
+ 로그인 기능
+ 회원가입 기능 전체
+ 직원리스트 기능 전체
+ 근태리스트 및 디테일 전체
## 설정파일
### LogIn
DB의 저장된 정보와의 비교를통해 저장된 정보와 입력된정보가 다르거나 존재하지않을시 로그인이 되지않으며 alert창이 보여진다
```java
script type="text/javascript">
		$('#login').click(function() {
			var id = $("input[type=text][name=id]").val()
			var password = $("input[type=password][name=password]").val()
			$.ajax({
				type : 'post',
				data : {
					'id' : id,
					'password' : password
				},
				url : '${contextPath}/login.do',
				success : function(data) {
					if (data == "1") {
						alert("아이디 또는 비밀번호가 다릅니다.");
					} else if (data == "2") {
						alert('로그인 성공');
						window.location.href = "home.do";
					}
				},
				error : function() {
					alert("로그인 에러");
				}
			});
		})
```
### Header
gtag로 화면전환시 로딩화면을 보여주었습니다.<br/>
출근버튼과 퇴근버튼클릭시 로그인한 사용자의 출근시간과 퇴근시간이 저장되도록 만들었습니다.<br/>
인사팀 직원만 보여주는 목록을 만들었습니다.
```java
<!-- Global site tag (gtag.js) - Google Analytics -->
<script async
	src="https://www.googletagmanager.com/gtag/js?id=UA-119386393-1"></script>
<script>
	window.dataLayer = window.dataLayer || [];
	function gtag() {
		dataLayer.push(arguments);
	}
	gtag('js', new Date());

	gtag('config', 'UA-119386393-1');
</script>

<!--출퇴근버튼 Html-->
<input style="display: none" class="btn btn-success" type="button" value="출근" id = "gowork"'>
<input style="display: none" class="btn btn-success" type="button" value="퇴근" id = "gohome" onclick='gotohome();'>
<!--출퇴근버튼 Script-->
$('#gowork').click(function() {
		$.ajax({
			type : 'post',
			url : '${contextPath}/insertattend.do',
			success : function(data){
				 alert(data); 
				 $.ajax({
						type : 'post',
						url : '${contextPath}/attendbutton.do',
						success : function(data){
							if(data == "0"){
								$('#gowork').css('display','block');
								$('#gohome').css('display','none');
							}else{
								$('#gohome').css('display','block');
								$('#gowork').css('display','none');
							}
						},
						error : function() {
							 alert("gowork button err"); 
						}
					})
			},
			error : function() {
				 alert("gowork err"); 
			}
		})
	}) 
	function gotohome() {
		$.ajax({
			type : 'post',
			url : '${contextPath}/updateattend.do',
			success : function(data){
				alert(data); 
				sessionStorage.clear();
				location.href="${contextPath}/";
			},
			error : function() {
				 alert("gohome err"); 
			}
		})
}

<!--인사팀목록-->
<c:if test="${dept_no eq '1'}">
	<li class="dropdown">
		<a href="javascript:;" class="dropdown-toggle">
			<span class="micon dw dw-library"></span><span class="mtext">근태관리</span>
		</a>
	<ul class="submenu">
		<li><a href="${contextPath}/attendancelist.do">근태 리스트</a></li>
	</ul>
		</li>
	</c:if>
	<c:if test="${dept_no ne '1'}">
	</c:if>
	<c:if test="${dept_no eq '1'}">
		<li class="dropdown">
			<a href="javascript:;" class="dropdown-toggle">
				<span class="micon dw dw-library"></span><span class="mtext">사원 관리</span>
			</a>
	<ul class="submenu">
		<li><a href="memberlist.do">사원 목록</a></li>
		<li><a href="insertmember">사원 추가</a></li>
	</ul>
		</li>
</c:if>
```
### MemberList
ajax를 통하여 선택한 부서별로 목록을 가지고오며 사원의 이름을 검색하여 DB에서 해당 직원의 데이터를 불러올수있다
```java
<--searchMemberList mapper-->
<select id="searchmemberlist" resultMap="resultMember" resultType="arraylist" parameterType="string">SELECT * FROM memberWHERE nameLIKE '%'||#{keyword}||'%'ORDER BY dept_no desc </select>
<--Html-->
<label><input class="deptrd" type="radio" name="dept_item" value="" checked="checked">모든부서</label>
<label><input class="deptrd" type="radio" name="dept_item" value="1">인사팀</label>
<label><input class="deptrd" type="radio" name="dept_item" value="2">경영팀</label>
<label><input class="deptrd" type="radio" name="dept_item" value="3">개발팀</label>
<input type="text" name="keyword" placeholder="이름입력">
<input type="button" name="search" value="검색">
<--Script-->
$(document).ready(function() {
		var deptno = "";
		$.ajax({
			type : 'post',
			url : '${contextPath}/deptmemberlist.do',
			data : {'dept' : deptno},
			dataType : 'json',
			success : function(data){
				$(".results").empty();
				for(var i=0; i<data.length; i++ ){
					var dept_name = data[i].dept_no;
					if(dept_name == 1){
						dept_name = '인사팀';
						}else if(dept_name == 2){
							dept_name='경영팀';
						}else{
							dept_name="개발팀";
						}
					console.log(dept_name);
					
					$(".results").append('<tr><td>'+ data[i].name +'</td>'+				
							'<td>'+dept_name+'</td>'+	
							'<td>'+data[i].position+'</td>'+
							'<td>'+data[i].phone+'</td>'+
							'<td>'+data[i].employ_date+'</td>'+
							'<td><a class="dropdown-item" href="${contextPath}/deletemember.do?id='+data[i].id+'"><i class="dw dw-delete-3"></i> Delete</a>'+
							'</td></tr>')
				}
			},
			error : function() {
				 alert("restController err"); 
			}
		});
	
		
	$("input[type=radio][name=dept_item]").click(function() {
		 deptno = $(this).val();
		console.log("로그는" + deptno);
		$.ajax({
			type : 'post',
			url : '${contextPath}/deptmemberlist.do',
			data : {'dept' : deptno},
			dataType : 'json',
			success : function(data){
				$(".results").empty();
				for(var i=0; i<data.length; i++ ){
					var dept_name = data[i].dept_no;
					if(dept_name == 1){
						dept_name = '인사팀';
						}else if(dept_name == 2){
							dept_name='경영팀';
						}else{
							dept_name="개발팀";
						}
					console.log(dept_name);
					
					$(".results").append('<tr><td>'+ data[i].name +'</td>'+				
							'<td>'+dept_name+'</td>'+	
							'<td>'+data[i].position+'</td>'+
							'<td>'+data[i].phone+'</td>'+
							'<td>'+data[i].employ_date+'</td>'+
							'<td><a class="dropdown-item" href="${contextPath}/deletemember.do?id='+data[i].id+'"><i class="dw dw-delete-3"></i> Delete</a>'+
							'</td></tr>')
				}
			},
			error : function() {
				 alert("restController err"); 
			}
		});
	}) 
	$("input[type=button][name=search]").click(function() {
		var keyword = $("input[type=text][name=keyword]").val()
		console.log("로그는" + keyword);
		$.ajax({
			type : 'post',
			url : '${contextPath}/searchmemberlist.do',
			data : {'keyword' : keyword},
			dataType : 'json',
			success : function(data){
				$(".results").empty();
				for(var i=0; i<data.length; i++ ){
					var dept_name = data[i].dept_no;
					if(dept_name == 1){
						dept_name = '인사팀';
						}else if(dept_name == 2){
							dept_name='경영팀';
						}else{
							dept_name="개발팀";
						}
					console.log(dept_name);
					
					$(".results").append('<tr><td>'+ data[i].name +'</td>'+				
							'<td>'+dept_name+'</td>'+	
							'<td>'+data[i].position+'</td>'+
							'<td>'+data[i].phone+'</td>'+
							'<td>'+data[i].employ_date+'</td>'+
							'<td><a class="dropdown-item" href="${contextPath}/deletemember.do?id='+data[i].id+'"><i class="dw dw-delete-3"></i> Delete</a>'+
							'</td></tr>')
				}
			},
			error : function() {
				 alert("restController err"); 
			}
		});
	}) 
	});
```
### AttendanceList
근태리스트를 오늘, 일주일의 선택이 가능하며 데이트 피커를 사용하여 날짜를 선택하여 검색가능하다.
```java
<--Html-->
<div class="btn-group btn-group-toggle" data-toggle="buttons" >
<label class="btn btn-outline-secondary active">
<input type="radio" name="options" id="option1" autocomplete="off" value="1" checked> 오늘</label>
<label class="btn btn-outline-secondary">
<input type="radio" name="options" id="option2" autocomplete="off" value="2"> 일주일</label>
<label class="btn btn-outline-secondary">
<input type="radio" name="options" id="option3" autocomplete="off" value="3"> 날짜선택</label>
</div>
&nbsp;&nbsp;
<div class="btn-group btn-group-toggle disset" data-toggle="buttons" >
<input style="width: 100px; height: 40px;" id="datepicker1" name="project_std_date" placeholder=" Select Date" type="text" autocomplete="off">
</div>
<div class="btn-group btn-group-toggle disset" data-toggle="buttons" >
<input style="width: 100px; height: 40px;" id="datepicker2" name="project_end_date" placeholder=" Select Date" type="text" autocomplete="off">
</div>
<div class="btn-group btn-group-toggle disset" data-toggle="buttons" >
<button name="search" type="button" class="btn btn-secondary">검색</button>
</div>

<--Script-->
$("#datepicker").datepicker({
		language : 'ko'
	});
	datePickerSet($("#datepicker1"), $("#datepicker2"), true);
	function datePickerSet(sDate, eDate, flag) {
		if (!isValidStr(sDate) && !isValidStr(eDate) && sDate.length > 0
				&& eDate.length > 0) {
			var sDay = sDate.val();
			var eDay = eDate.val();
			if (flag && !isValidStr(sDay) && !isValidStr(eDay)) { 
				var sdp = sDate.datepicker().data("datepicker");
				sdp.selectDate(new Date(sDay.replace(/-/g, "/"))); 
				var edp = eDate.datepicker().data("datepicker");
				edp.selectDate(new Date(eDay.replace(/-/g, "/"))); 
			}
			if (!isValidStr(eDay)) {
				sDate.datepicker({
					maxDate : new Date(eDay.replace(/-/g, "/"))
				});
			}
			sDate.datepicker({
				language : 'en',
				dateFormat : 'yyyy-mm-dd',
				autoClose : true,
				onSelect : function() {
					datePickerSet(sDate, eDate);
				}
			});
			if (!isValidStr(sDay)) {
				eDate.datepicker({
					minDate : new Date(sDay.replace(/-/g, "/"))
				});
			}
			eDate.datepicker({
				language : 'en',
				dateFormat : 'yyyy-mm-dd',
				autoClose : true,
				onSelect : function() {
					datePickerSet(sDate, eDate);
				}
			});
		} else if (!isValidStr(sDate)) {
			var sDay = sDate.val();
			if (flag && !isValidStr(sDay)) { 
				var sdp = sDate.datepicker().data("datepicker");
				sdp.selectDate(new Date(sDay.replace(/-/g, "/")));
			}
			sDate.datepicker({
				language : 'en',
				dateFormat : 'yyyy-mm-dd',
				autoClose : true
			});
		}
            function isValidStr(str) {
                if (str == null || str == undefined || str == "")
                    return true;
                else
                    return false;
            }
        }
```
### AttendanceDetail
fullCalendar 를 이용하여 해당달을 선택할수있으며 해당달의  출근시간과 퇴근시간을 일별로 보여준다.<br/>
Jstl을 이용하여 Footer에 DB에 저장된 이번달의 출근시간과 오늘의 출퇴근시간을 보여준다.
```java
<--Script-->
(function () {
		jQuery(function() {
			// page is ready
			jQuery('#calendar').fullCalendar({
				themeSystem: 'bootstrap4',
				// emphasizes business hours
				businessHours: false,
				defaultView: 'month',
				// event dragging & resizing
				editable: false,
				displayEventTime : false,
				// header
				header: {
					left: '',
					center: 'title',
					right: 'today prev,next'
				},
				events: [
					<c:forEach var="vo" items="${list}">
				{
					title : "출근시간 ${vo.attend_gotowork}\n\n퇴근시간 ${vo.attend_gotohome}\n",
					start : "${vo.attend_work_date}",
					end : "${vo.attend_work_date}",
					backgroundColor : "#d5eff4",
					textColor : "#000000"
				},
					</c:forEach>
				],
				dayClick: function() {
					jQuery('#modal-view-event-add').modal();
				},
				eventClick: function(event, jsEvent, view) {
					jQuery('.event-icon').html("<i class='fa fa-"+event.icon+"'></i>");
					jQuery('.event-title').html(event.title);
					jQuery('.event-body').html(event.description);
					jQuery('.eventUrl').attr('href',event.url);
					jQuery('#modal-view-event').modal();
				},
			})
	});
	})(jQuery);
 
 <--Html-->
 <c:if test="${not empty avgString }">
	<div class="lower_bar">이번달 근무시간 : ${avgString} 시간</div>
</c:if>
<c:if test="${ empty avgString }">
</c:if>
<c:forEach items="${today}" var="today">
	<div class="lower_bar">오늘 출근시간 : ${today.attend_gotowork}</div>
	<div class="lower_bar">오늘 퇴근시간 : ${today.attend_gotohome}</div>
</c:forEach>
<c:if test="${not empty worktime }">
	<div class="lower_bar">오늘 근무시간 : ${worktime}</div>
</c:if>	
<c:if test="${empty worktime }">
	<div class="lower_bar">출근 또는 퇴근되지 않았습니다.</div>
</c:if>
```
