---
author: Tomasz Dziurko
comments: true
date: 2012-10-11 15:00:47+00:00
layout: post
slug: fixing-bug-hibernate-envers
title: Fixing bug in Hibernate Envers
wordpress_id: 1667
categories:
- Java
tags:
- Envers
- transaction rollback
---

Recently in our project we were reported a strange bug. In a one report where we display historical data provided by [Hibernate Envers](http://www.jboss.org/envers), users encountered duplicated records in dropdown used for filtering. We tried to find a source of this bug, but after spending a few hours looking at the code responsible for this functionality we had to give up and ask for a dump from production database to check what actually is stored in one table. And when we got it and started investigating, it turned out that there is a bug in Hibernate Envers 3.6 that is a cause of our problems. But luckily after some investigation and invaluable help from [Adam Warski](http://www.warski.org/blog/) (author of Envers) we were able to fix this issue.
<!-- more -->


### Bug itself


Let's consider following scenario:



	
  1. Transaction is started. We insert some audited entities during it and then it is rolled back.

	
  2. The same EntityManager is reused to start another transaction

	
  3. Second transaction is committed


But when we check audit tables for entities that were created and then rolled back in step one we will notice that they are still there and were not rolled back as we could expect. We were able to reproduce it in a failing test in our project so next step was to prepare failing test in Envers so we could verify if our fix is working.


### Failing test


The simplest test cases already present in Envers are located in Simple.java class and they look quite straightforward:

[java]
public class Simple extends AbstractEntityTest {
    private Integer id1;

    public void configure(Ejb3Configuration cfg) {
        cfg.addAnnotatedClass(IntTestEntity.class);
    }

    @Test
    public void initData() {
        EntityManager em = getEntityManager();
        em.getTransaction().begin();
        IntTestEntity ite = new IntTestEntity(10);
        em.persist(ite);
        id1 = ite.getId();
        em.getTransaction().commit();

        em.getTransaction().begin();
        ite = em.find(IntTestEntity.class, id1);
        ite.setNumber(20);
        em.getTransaction().commit();
    }

    @Test(dependsOnMethods = "initData")
    public void testRevisionsCounts() {
        assert Arrays.asList(1, 2).equals(getAuditReader().getRevisions(IntTestEntity.class, id1));
    }

    @Test(dependsOnMethods = "initData")
    public void testHistoryOfId1() {
        IntTestEntity ver1 = new IntTestEntity(10, id1);
        IntTestEntity ver2 = new IntTestEntity(20, id1);

        assert getAuditReader().find(IntTestEntity.class, id1, 1).equals(ver1);
        assert getAuditReader().find(IntTestEntity.class, id1, 2).equals(ver2);
    }
}
[/java]

so preparing my failing test executing scenario described above wasn't a rocket science:

[java]
/**
 * @author Tomasz Dziurko (tdziurko at gmail dot com)
 */
public class TransactionRollbackBehaviour  extends AbstractEntityTest {

    public void configure(Ejb3Configuration cfg) {
        cfg.addAnnotatedClass(IntTestEntity.class);
    }

    @Test
    public void testAuditRecordsRollback() {
        // Given
        EntityManager em = getEntityManager();
        em.getTransaction().begin();
        IntTestEntity iteToRollback = new IntTestEntity(30);
        em.persist(iteToRollback);
        Integer rollbackedIteId = iteToRollback.getId();
        em.getTransaction().rollback();

        // When
        em.getTransaction().begin();
        IntTestEntity ite2 = new IntTestEntity(50);
        em.persist(ite2);
        Integer ite2Id = ite2.getId();
        em.getTransaction().commit();

        // Then
        List<Number> revisionsForSavedClass = getAuditReader().getRevisions(IntTestEntity.class, ite2Id);
        assertEquals(revisionsForSavedClass.size(), 1, "There should be one revision for inserted entity");

        List<Number> revisionsForRolledbackClass = getAuditReader().getRevisions(IntTestEntity.class, rollbackedIteId);
        assertEquals(revisionsForRolledbackClass.size(), 0, "There should be no revisions for insert that was rolled back");
    }
}

[/java]

Now I could verify that tests is failing on forked [3.6 branch](https://github.com/hibernate/hibernate-orm/tree/3.6) and check if fix that we had is making this test green.


### The fix


After writing failing test in our project, I placed several breakpoints in Envers code to understand better what is wrong there. But imagine being thrown in a project developed for a few years by many programmers smarter than you. I felt overwhelmed and have no idea where fix should be applied and what exactly is not working as expected. Luckily in [my company](http://softwaremill.com) we have Adam Warski on board. He is the initial author of Envers and actually he pointed us the solution.

The fix itself contains only check that registers audit process that will be execute on transaction completion only when such process is still in the _Map<Transaction, AuditProcess>_ for given transaction. It sounds complicated, but if you look at the class _AuditProcessManager_ in [this commit](https://github.com/tdziurko/hibernate-orm/commit/02c071718f29797bf0d76f3a6f2e6a5438ccaab1) it should be more clear what is happening there.


### Official path


Besides locating problem and fixing it, there are some more official steps that must be performed to have fix included in Envers.

Step 1. Create JIRA issue with bug - [https://hibernate.onjira.com/browse/HHH-7682](https://hibernate.onjira.com/browse/HHH-7682)

Step 2: Create local branch _Envers-bugfix-HHH-7682_ of forked Hibernate 3.6

Step 3: Commit and push failing test and fix to your local and remote repository on Github

Step 4: Create pull request - [https://github.com/hibernate/hibernate-orm/pull/393](https://github.com/hibernate/hibernate-orm/pull/393)

Step 5: Wait for merge

And that's all. Now fix is merged into main repository and we have one bug less in the world of open source :)
