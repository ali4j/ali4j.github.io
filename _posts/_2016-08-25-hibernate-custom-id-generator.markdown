---
layout: post
title: "Hibernate custom id generator"
date: 2016-8-25 00:00:00 +0400
description: generating custom ids for entities in hibernate. # Add post description (optional)
img: hibernate_custom_id_generator.jpg # Add image post (optional)
tags: [logging,java,hibernate, jpa, id]
---

Generating a custom id for the entities while using hibernate is quite simple. 
In the scenario I faced I had to use a id generator that starts from a specific id value, I had some records previously inserted into the table with their own id value and for this reason I needed to create a custom id generator for the entities.

Creating an id generator is straightforward and includes 3 steps:

1- First you need to add a sequence to the database (in my scenario I had to use a sequence in sql server 2012).
2- org.hibernate.id.IdentifierGenerator has to be implemeneted and a body for generate method should be provided.
3- newly created id generator can be used as the strategy in @GenericGenerator annotation.

This is the first step, sequnce cration actually:

{% highlight SQL %}
CREATE SEQUENCE [dbo].[Custom_Sequence]
AS [BIGINT]
START WITH 61000
INCREMENT BY 1
MINVALUE -9223372036854775808
MAXVALUE 9223372036854775808
CACHE
{% endhighlight %}

second step includes implementing IdentifierGenerator:

{% highlight JAVA %}
public class NewDataIdGenerator implements IdentifierGenerator{
	private static final Log logger = LogFactory.getLog(NewDataIdGenerator.class);
	@Override
	public Serializable generate(SessionImplementor session, Object obj) throws HibernateException {
		Connection con = session.connection();
		Long nextValue = null;
		try {
			PreparedStatement p = con.prepareStatement("SELECT NEXT VALUE FOR Custom_Sequence as nextVal");
			ResultSet rs = p.executeQuery();
			while(rs.next()) {
				nextValue = rs.getLong("nextVal");
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		if(logger.isDebugEnabled()) logger.debug("new id is generated:" + nextValue);
		return nextValue;
	}
}
{% endhighlight %}

and the last step is using custom generator as a strategy in the entity:

{% highlight JAVA %}
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import org.hibernate.annotations.GenericGenerator;

@Entity
public class CustomEntity {
	@Id
	@GenericGenerator(name="seq_id", strategy="ir.gov.tax.services.bita.exposed.sendcustomsdeclarationserialnumberservice.idgenerator.NewDataIdGenerator")
	@GeneratedValue(generator="seq_id")
	private Long id;
	/*getter & setter*/
}
{% endhighlight %}


Important point that has to be kept in mind is that with the usage of Hibernate classes (not jpa api) the code gets dependent to the framework and the maintenance of the code will face problems.