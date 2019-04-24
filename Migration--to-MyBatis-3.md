## TypeAlias moved to the config file.

* TypeAlias should be defined in sql-map-config instead of mapper itself.
*   All the type TypeAlias will be extracted tot  seperate _TypeAlias.text file by the converter. 
New header:

The ibatis header:
```
<!DOCTYPE sqlMap PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN" "http://ibatis.apache.org/dtd/sql-map-2.dtd">
```
is changed to 
```
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

The root node `<sqlMap>` is changed to  `<mapper>`


##`<statement>`
Change to use `<select>, <insert>, <update>, <delete> `by inspecting the SQL inside

## `<select>` useCache by default is true 
set useCache="false" for select <select> 

##parameterMap is still supported
"class" attribute is changed to "type"
`<parameterMap id="addAdminBookmarkMap" class="AdminBookmarkVO">`
changed to 
`<parameterMap id="addAdminBookmarkMap" type="AdminBookmarkVO">`

## parameterClass change to parameterType
`parameterClass="map" changed to parameterType="map"`

## dynamic `<isEqual>, <isNotEqual>, <isGreaterThan>, <isGreaterEqual>,<isLessEqual>`
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE sqlMap PUBLIC "-//ibatis.apache.org//DTD SQL Map 2.0//EN" "http://ibatis.apache.org/dtd/sql-map-2.dtd">
<sqlMap>
    <select id="dynamicGetAccountList" parameterClass="Account" resultMap="account-result">
        select * from ACCOUNT
        <dynamic prepend="WHERE">
            <isNotNull prepend="AND" property="FirstName" open="(" close=")">
                ( ACC_FIRST_NAME = #FirstName#
                <isNotNull prepend="OR" property="LastName">
                    ACC_LAST_NAME = #LastName#
                </isNotNull>
                )
            </isNotNull>
            <isNotNull prepend="AND" property="EmailAddress">
                ACC_EMAIL like #EmailAddress#
            </isNotNull>
            <isGreaterThan prepend="AND" property="Id" compareValue="0">
                ACC_ID = #Id#
            </isGreaterThan>

            <isEqual prepend="AND" property="status" compareValue="Y">
                MARRIED = ‘TRUE'
            </isEqual>

            <isNotEqual prepend="AND"
                        property="status"
                        compareValue="N">
                MARRIED = ‘FALSE'
            </isNotEqual>

            <isGreaterThan prepend="AND"
                           property="age"
                           compareValue="18">
                ADOLESCENT = ‘FALSE'
            </isGreaterThan>

            <isGreaterEqual prepend="AND"
                            compareProperty="shoeSize"
                            compareValue="12">
                BIGFOOT = ‘TRUE'
            </isGreaterEqual>

            <isLessEqual prepend="AND"
                         property="age"
                         compareValue="18">
                ADOLESCENT = ‘TRUE'
            </isLessEqual>

            <isPropertyAvailable property="id" >
                ACCOUNT_ID=#id#
            </isPropertyAvailable>

            <isNotPropertyAvailable property="age" >
                STATUS='New'
            </isNotPropertyAvailable>

            <isNull prepend="AND" property="order.id" >
                ACCOUNT.ACCOUNT_ID = ORDER.ACCOUNT_ID(+)
            </isNull>

            <isNotEmpty property="firstName" >
                LIMIT 0, 20
            </isNotEmpty>

            <isNotEmpty prepend="AND" property="firstName" >
                FIRST_NAME LIKE '%$FirstName$%'
            </isNotEmpty>

        </dynamic>
        order by ACC_LAST_NAME
    </select>
/sqlMap>
```
after conversion:
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE mapper PUBLIC "http://mybatis.org/dtd/mybatis-3-mapper.dtd" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper>
    <select id="dynamicGetAccountList" parameterType="Account" resultMap="account-result">
        select * from ACCOUNT
        WHERE
        <if test="FirstName != null ">
            ( AND ( ACC_FIRST_NAME = #{FirstName}
            <if test="LastName != null ">
                OR  ACC_LAST_NAME = #{LastName}
            </if>
            )
            )</if>
        <if test="EmailAddress != null ">
            AND  ACC_EMAIL like #{EmailAddress}
        </if>
        <if test="Id &gt; 0">
            AND  ACC_ID = #{Id}
        </if>

        <if test="status == Y">
            AND  MARRIED = ‘TRUE'
        </if>

        <if test="status &lt;&gt; N">
            AND  MARRIED = ‘FALSE'
        </if>

        <if test="age &gt; 18">
            AND  ADOLESCENT = ‘FALSE'
        </if>

        <if test="shoeSize &gt;= 12">
            AND  BIGFOOT = ‘TRUE'
        </if>

        <if test="age &lt; 18">
            AND  ADOLESCENT = ‘TRUE'
        </if>

        <if test="id != null ">
            ACCOUNT_ID=#{id}
        </if>

        <if test="age == null ">
            STATUS='New'
        </if>

        <if test="order.id == null ">
            AND  ACCOUNT.ACCOUNT_ID = ORDER.ACCOUNT_ID(+)
        </if>

        <if test=" firstName != null and firstName != '' ">
            LIMIT 0, 20
        </if>

        <if test=" firstName != null and firstName != '' ">
            AND  FIRST_NAME LIKE '%$FirstName$%'
        </if>
        order by ACC_LAST_NAME
    </select>
</mapper>
```


## dynamic iterate changes to "foreach"
before:
```
<dynamic prepend="IN">
   <iterate property="appAdminIDs" open="(" close=")" conjunction=",">
    #appAdminIDs[]#
  </iterate>
</dynamic>
```
After:
```
IN
<foreach item="item" index="index" collection="appAdminIDs" open="(" separator="," close=")">
    #{item}
</foreach>
```

## "class" attribute in resultMap changed to "type"
new:
```
<resultMap type="AdminQuickCommand" id="adminQuickCommandsResult">
```


## "groupBy" in the resultMap
```
<resultMap id="StudentCustomColResultMap" type="StudentVOX" groupBy="studentID">
 <result property="studentID" column="stud_id" jdbcType="VARCHAR"/>
 result property="studentCustomColumnList" resultMap="Student.StudentCustomColumnMap" />
</resultMap>
```
change to 
```
<resultMap id="StudentCustomColResultMap" type="StudentVOX">
   <id property="studentID" column="stud_id" jdbcType="VARCHAR"/>
   <collection column="stud_id" property="studentCustomColumnList" resultMap="Student.StudentCustomColumnMap"/>
</resultMap>
```

### another example

convert 
```
  <resultMap extends="result_HomeCustomTileVO" groupBy="id" id="result_HomeCustomTileVOX" class="HomeCustomTileVOX">
        <result property="homeCustomTileLocaleVOs" resultMap="GroupByExample.result_HomeCustomTileLocaleVO"/>
    </resultMap>

    <resultMap extends="result_CoverPageVOX" groupBy="coverPageID" id="result_CoverPageDataVOX" class="CoverPageDataVOX">
        <result property="customTileVOXs" resultMap="GroupByExample.result_HomeCustomTileVOX"/>
    </resultMap>
```

to

```
  <resultMap id="result_CoverPageVO" type="CoverPageVO">
        <id column="COVER_PAGE_ID" jdbcType="VARCHAR" property="coverPageID"/>
        <result column="LAYOUT_TYPE_ID" jdbcType="VARCHAR" property="layoutTypeID"/>
        <result column="ACTIVE" jdbcType="boolean" property="active"/>
        <result column="LAYOUT" jdbcType="CLOB" property="layout"/>
        <result column="LST_UPD_USR" jdbcType="VARCHAR" property="lastUpdateUser"/>
        <result column="LST_UPD_TSTMP" jdbcType="DATE" property="lastUpdateTimestamp"/>
    </resultMap>

    <resultMap extends="result_CoverPageVO" id="result_CoverPageVOX" type="CoverPageVOX">
    </resultMap>
  <resultMap id="result_HomeCustomTileVO" type="HomeCustomTileVO">
        <id column="custom_tile_id" jdbcType="varchar" property="id"/>
        <result column="description_labelID" jdbcType="varchar" property="description"/>
        <result column="title_labelID" jdbcType="varchar" property="title"/>
        <result column="content_labelID" jdbcType="varchar" property="content"/>
        <result column="custom_tile_type" jdbcType="varchar" property="type"/>
        <result column="custom_tile_name" jdbcType="varchar" property="name"/>
        <result column="always_active" jdbcType="boolean" property="isAlwaysActive"/>
        <result column="active_start_date" jdbcType="date" property="activeStartDate"/>
        <result column="active_end_date" jdbcType="date" property="activeEndDate"/>
        <result column="COLUMN_SIZE" jdbcType="NUMERIC" property="columnSize"/>
        <result column="ROW_SIZE" jdbcType="NUMERIC" property="rowSize"/>
    </resultMap>

    <resultMap extends="result_HomeCustomTileVO" id="result_HomeCustomTileVOX" type="HomeCustomTileVOX">
        <!--groupbBy id-->
        <collection column="custom_tile_id" property="homeCustomTileLocaleVOs" resultMap="GroupByExample.result_HomeCustomTileLocaleVO"/>
    </resultMap>

    <resultMap extends="result_CoverPageVOX" id="result_CoverPageDataVOX" type="CoverPageDataVOX">
        <!--groupbBy coverPageID-->
        <collection column="COVER_PAGE_ID" property="customTileVOXs" resultMap="GroupByExample.result_HomeCustomTileVOX"/>
    </resultMap>

```
### Use collections
```
 <result property="expectedValue" column="ATTR_EXPECTED_VALUE"/>	    
	    <collection property="trdReturnValues" ofType="com.myorg.test.TestResultDetailReturnValue">
	      <id property="id" column="X_TEST_RESULT_DETAIL_RV_ID"/>
	      <result property="returnName" column="RETURN_NAME"/>
	      <result property="returnValue" column="RETURN_VALUE"/> 
	    </collection>
```
## Inline parameters
* `#value#` `is now:` `#{value}`
* `#domainID:VARCHAR#`  is  changed to    `#{domainID,jdbcType=VARCHAR}`
* $orderBy$  change to ${orderBy}

## jdbcType changes
jdbcType="ORACLECURSOR" is now: jdbcType="CURSOR"
and
jdbcType="NUMBER" is now: jdbcType="NUMERIC"


## isEqual 

`<isEqual property="catalogQueryType" compareValue="FILTERED_CATALOG_SEARCH">`
`<include refid="FILTERED_CATALOG_SEARCH"/>`
`</isEqual>`

changeto 

`<if test="catalogQueryType =='FILTERED_CATALOG_SEARCH' ">`
`<include refid="FILTERED_CATALOG_SEARCH"/>`
`</if>`

## To check empty  string
` <if test="title != null and title != ''">`
    `AND title like #{title}`
  `</if>`


##isPropertyAvailable
```
<isPropertyAvailable property="displayPriceType" >
<![CDATA[ 
desc
]]>
</isPropertyAvailable>
```
change to to
```
<if test="displayPriceType!=null">
  <![CDATA[
    desc
   ]]>
</if>
```

##if test examples:

'<if test="_parameter.containsKey('clusterId')"> '


##Procedure

```
<parameterMap id="studentCurriculumStatusParamMap" type="map">
	<parameter javaType="java.lang.String" jdbcType="VARCHAR" mode="IN" property="studentID"/>
	<parameter javaType="java.lang.String" jdbcType="VARCHAR" mode="IN" property="curriculumID"/>
	<parameter javaType="java.lang.String" jdbcType="VARCHAR" mode="IN" property="rootCurriculumID"/>
	<parameter javaType="java.lang.String" jdbcType="VARCHAR" mode="OUT" property="curriculumStatus"/>
	<parameter javaType="java.sql.Timestamp" jdbcType="DATE" mode="OUT" property="expirationDate"/>
	<parameter javaType="java.lang.Integer" jdbcType="NUMERIC" mode="OUT" property="remainingDays"/>
	<parameter javaType="java.sql.Timestamp" jdbcType="DATE" mode="OUT" property="nextActionDate"/>
</parameterMap>


 <procedure id="getStudentCurriculumStatus" parameterMap="studentCurriculumStatusParamMap">
	        {call pkg_student.stud_qual_status(?, ?, ?,?, ?, ?, ?)}
 </procedure>
```
change to:
```
<update id="getStudentCurriculumStatus" statementType="CALLABLE" parameterMap="studentCurriculumStatusParamMap">
		{call pkg_student.stud_qual_status(?, ?, ?,?, ?, ?, ?)}
</update>
```


Other Exmaples
```
<resultMap id="UserResult" type="User">
    <id property="userId" column="userId"/>
    <result property="firstName" column="firstName"/>
    <result property="lastName" column="lastName"/>     
</resultMap>
In your select statement, change the parameter type to java.util.Map.

<select id="getUsers" statementType="CALLABLE" parameterType="java.util.Map"> 
    {call GetUsers(#{users, jdbcType=CURSOR, javaType=java.sql.ResultSet, mode=OUT, resultMap=UserResult})} 
</select>

```
```
<update id="someProcedure" statementType="CALLABLE">
    {call some procedure(
            #{someInParamA, mode=IN},
            #{someInParamB, jdbcType=ARRAY, mode=IN},
            #{someOutParamA, javaType=Boolean, jdbcType=NUMERIC, mode=OUT },
            #{someOutParamB, javaType=Object, jdbcType=ARRAY, jdbcTypeName=SOMEJDBCTYPE, mode=OUT})}
</update>
```
So to get the out parameters, it would look something like this.
```
mapper.getSomeProcedure(someBean);
//out params populated on the object passed to the mapper :)
String outA = bean.getSomeOutParamA();
```