# Production Incidents & Solutions - Real-World Stories

> **"The best engineers learn from production incidents, not just from books."**

This guide contains real production incidents from top tech companies, how they were solved, and lessons learned. Essential for staff engineer interviews and understanding real-world system behavior.

---

## Table of Contents

1. [High Availability Incidents](#high-availability-incidents)
2. [Performance Degradation Incidents](#performance-degradation-incidents)
3. [Data Loss Incidents](#data-loss-incidents)
4. [Security Incidents](#security-incidents)
5. [Scaling Incidents](#scaling-incidents)
6. [Lessons Learned](#lessons-learned)

---

## High Availability Incidents

### Incident 1: AWS S3 Outage (2017)

**What Happened:**
- AWS S3 (Simple Storage Service) experienced a 4-hour outage
- Affected thousands of services including Netflix, Slack, Trello
- Root cause: Human error during debugging caused removal of too many servers

**Impact:**
- Millions of users affected
- Estimated $150M+ in lost revenue
- Services completely down

**Root Cause:**
1. Engineer ran command to remove servers from one S3 subsystem
2. Typo in command removed servers from multiple subsystems
3. System couldn't recover fast enough
4. Cascading failures

**Solution:**
1. **Immediate**: Manually restart affected systems
2. **Short-term**: Add safeguards to prevent similar mistakes
3. **Long-term**: 
   - Better automation for recovery
   - More redundancy
   - Improved monitoring

**Lessons Learned:**
- ✅ Always have safeguards for destructive operations
- ✅ Test recovery procedures regularly
- ✅ Human errors are inevitable - design for them
- ✅ Monitoring and alerting are critical

**Interview Question:**
"How would you prevent a similar incident?"

**Answer:**
1. **Safeguards**: Require confirmation for destructive operations
2. **Rate Limiting**: Limit number of servers that can be removed at once
3. **Dry Run Mode**: Test commands in dry-run mode first
4. **Automated Recovery**: Auto-recovery scripts
5. **Better Monitoring**: Alert on unusual patterns
6. **Chaos Engineering**: Regularly test failure scenarios

---

### Incident 2: GitHub Database Outage (2018)

**What Happened:**
- GitHub experienced 24-hour outage
- Database primary failed, failover didn't work properly
- Manual intervention required

**Impact:**
- All GitHub services down
- Millions of developers affected
- Lost productivity worldwide

**Root Cause:**
1. Database primary server failed
2. Automatic failover didn't trigger
3. Manual failover took hours
4. Data consistency issues during recovery

**Solution:**
1. **Immediate**: Manual database failover
2. **Short-term**: Fix automatic failover mechanism
3. **Long-term**:
   - Improve database replication
   - Better monitoring
   - Regular failover testing

**Lessons Learned:**
- ✅ Test failover procedures regularly
- ✅ Don't rely solely on automatic failover
- ✅ Have manual procedures documented
- ✅ Monitor replication lag

**Interview Question:**
"How would you design a database failover system?"

**Answer:**
1. **Health Checks**: Continuous monitoring of primary
2. **Automatic Failover**: Promote replica on primary failure
3. **Data Consistency**: Ensure replica is up-to-date
4. **Split-Brain Prevention**: Only one primary at a time
5. **Monitoring**: Alert on replication lag
6. **Testing**: Regular failover drills

---

### Incident 3: Facebook Outage (2021)

**What Happened:**
- Facebook, Instagram, WhatsApp down for 6 hours
- Root cause: BGP (Border Gateway Protocol) configuration error
- DNS couldn't resolve Facebook domains

**Impact:**
- 3.5 billion users affected
- Estimated $60M+ in lost revenue
- Global impact

**Root Cause:**
1. Engineer made BGP configuration change
2. Removed routes to Facebook's DNS servers
3. Internet couldn't find Facebook's servers
4. Cascading failures

**Solution:**
1. **Immediate**: Revert BGP configuration
2. **Short-term**: Add safeguards for BGP changes
3. **Long-term**: Better change management

**Lessons Learned:**
- ✅ Network changes are high-risk
- ✅ Need approval process for critical changes
- ✅ Test changes in staging first
- ✅ Have rollback procedures ready

---

## Performance Degradation Incidents

### Incident 4: Twitter "Fail Whale" (2008-2010)

**What Happened:**
- Twitter frequently down during high traffic events
- "Fail Whale" error page shown to users
- Database couldn't handle load

**Impact:**
- Frequent outages
- Poor user experience
- Lost users to competitors

**Root Cause:**
1. Monolithic Ruby on Rails application
2. Single database couldn't scale
3. No caching strategy
4. Synchronous operations blocking

**Solution:**
1. **Short-term**: Add caching (Memcached)
2. **Medium-term**: Database sharding
3. **Long-term**: 
   - Move to microservices
   - Use message queues
   - Better caching strategy

**Lessons Learned:**
- ✅ Design for scale from day one
- ✅ Caching is critical
- ✅ Async processing for non-critical operations
- ✅ Monitor and optimize database queries

**Interview Question:**
"How would you scale Twitter's architecture?"

**Answer:**
1. **Caching**: Cache tweets, user profiles, timelines
2. **Database Sharding**: Shard by user_id
3. **Read Replicas**: Separate read and write databases
4. **Message Queues**: Async processing for non-critical operations
5. **CDN**: Serve static content from edge
6. **Microservices**: Split into services (tweets, users, timelines)

---

### Incident 5: Reddit "503 Service Unavailable" (2015)

**What Happened:**
- Reddit experienced frequent downtime
- Database queries taking too long
- Application servers timing out

**Impact:**
- Users couldn't access Reddit
- Poor user experience
- Lost traffic

**Root Cause:**
1. N+1 query problem
2. Missing database indexes
3. No query optimization
4. Synchronous operations

**Solution:**
1. **Immediate**: Add database indexes
2. **Short-term**: Optimize queries, add caching
3. **Long-term**:
   - Database read replicas
   - Query optimization
   - Better monitoring

**Lessons Learned:**
- ✅ Database indexes are critical
- ✅ Monitor slow queries
- ✅ Optimize queries before scaling
- ✅ Use connection pooling

---

## Data Loss Incidents

### Incident 6: GitLab Data Deletion (2017)

**What Happened:**
- GitLab accidentally deleted production database
- 6 hours of data lost
- Only backup was 6 hours old

**Impact:**
- Lost 6 hours of user data
- Service down for hours
- Trust issues

**Root Cause:**
1. Engineer ran `rm -rf` on wrong server
2. Deleted production database
3. Backup was 6 hours old
4. No recent backup available

**Solution:**
1. **Immediate**: Restore from 6-hour-old backup
2. **Short-term**: Manual data recovery from logs
3. **Long-term**:
   - Automated backups every hour
   - Multiple backup locations
   - Test restore procedures

**Lessons Learned:**
- ✅ Always have recent backups
- ✅ Test backup and restore procedures
- ✅ Multiple backup locations
- ✅ Safeguards for destructive operations
- ✅ Regular backup testing

**Interview Question:**
"How would you design a backup strategy?"

**Answer:**
1. **Frequency**: Hourly backups for production
2. **Retention**: 30 days daily, 12 months monthly
3. **Locations**: Multiple geographic locations
4. **Testing**: Regular restore tests
5. **Automation**: Automated backup process
6. **Monitoring**: Alert on backup failures
7. **Encryption**: Encrypt backups
8. **Versioning**: Keep multiple versions

---

### Incident 7: MongoDB Ransomware Attack (2017)

**What Happened:**
- Thousands of MongoDB databases hacked
- Data deleted, ransom demanded
- No backups available

**Impact:**
- Complete data loss
- Service down
- Reputation damage

**Root Cause:**
1. Databases exposed to internet
2. No authentication enabled
3. No backups
4. Vulnerable to attacks

**Solution:**
1. **Immediate**: Secure databases
2. **Short-term**: Enable authentication
3. **Long-term**:
   - Regular backups
   - Network security
   - Access controls

**Lessons Learned:**
- ✅ Never expose databases to internet
- ✅ Always enable authentication
- ✅ Regular backups are critical
- ✅ Network security is important
- ✅ Principle of least privilege

---

## Security Incidents

### Incident 8: Equifax Data Breach (2017)

**What Happened:**
- 147 million users' data breached
- Social security numbers, addresses exposed
- Apache Struts vulnerability exploited

**Impact:**
- Massive data breach
- Legal issues
- Reputation damage
- $700M+ in costs

**Root Cause:**
1. Apache Struts vulnerability not patched
2. No security monitoring
3. Breach went undetected for months
4. Poor security practices

**Solution:**
1. **Immediate**: Patch vulnerabilities
2. **Short-term**: Security audit
3. **Long-term**:
   - Regular security updates
   - Security monitoring
   - Penetration testing
   - Security training

**Lessons Learned:**
- ✅ Keep software updated
- ✅ Security monitoring is critical
- ✅ Regular security audits
- ✅ Patch management process
- ✅ Incident response plan

---

### Incident 9: Uber Data Breach (2016)

**What Happened:**
- 57 million users' data breached
- Hackers accessed database
- Company tried to cover it up

**Impact:**
- Data breach
- Legal issues
- Reputation damage
- CEO resigned

**Root Cause:**
1. GitHub repository with credentials exposed
2. Credentials used to access database
3. No proper access controls
4. Poor security practices

**Solution:**
1. **Immediate**: Change all credentials
2. **Short-term**: Security audit
3. **Long-term**:
   - Secure credential management
   - Access controls
   - Security monitoring
   - Incident response plan

**Lessons Learned:**
- ✅ Never commit credentials to code
- ✅ Use secret management (AWS Secrets Manager, HashiCorp Vault)
- ✅ Principle of least privilege
- ✅ Security monitoring
- ✅ Transparent incident response

---

## Scaling Incidents

### Incident 10: Amazon Prime Day (2019)

**What Happened:**
- Amazon Prime Day traffic spike
- Some services couldn't handle load
- Checkout failures

**Impact:**
- Lost sales
- Poor user experience
- Reputation damage

**Root Cause:**
1. Underestimated traffic
2. Some services not scaled properly
3. Database bottlenecks
4. Cache misses

**Solution:**
1. **Immediate**: Scale up services
2. **Short-term**: Add more capacity
3. **Long-term**:
   - Better capacity planning
   - Auto-scaling
   - Load testing
   - Caching strategy

**Lessons Learned:**
- ✅ Load test before major events
- ✅ Auto-scaling is critical
- ✅ Capacity planning
- ✅ Monitor and scale proactively
- ✅ Have scaling runbooks ready

**Interview Question:**
"How would you prepare for a traffic spike?"

**Answer:**
1. **Load Testing**: Test with expected load
2. **Auto-scaling**: Configure auto-scaling rules
3. **Caching**: Cache frequently accessed data
4. **CDN**: Use CDN for static content
5. **Database**: Scale databases (read replicas, sharding)
6. **Monitoring**: Monitor key metrics
7. **Runbooks**: Have scaling procedures ready
8. **Capacity Planning**: Estimate required capacity

---

### Incident 11: Pokemon Go Launch (2016)

**What Happened:**
- Pokemon Go launched with massive popularity
- Servers couldn't handle load
- Frequent outages

**Impact:**
- Service unavailable
- Poor user experience
- Lost users

**Root Cause:**
1. Underestimated popularity
2. Servers not scaled properly
3. Database bottlenecks
4. No auto-scaling

**Solution:**
1. **Immediate**: Add more servers
2. **Short-term**: Database scaling
3. **Long-term**:
   - Auto-scaling
   - Better capacity planning
   - Load testing

**Lessons Learned:**
- ✅ Plan for unexpected popularity
- ✅ Auto-scaling is essential
- ✅ Load test before launch
- ✅ Have scaling procedures ready
- ✅ Monitor and react quickly

---

## Lessons Learned

### Common Patterns

1. **Human Error**
   - Most incidents caused by human error
   - Need safeguards and automation
   - Design for human mistakes

2. **Lack of Monitoring**
   - Many incidents went undetected
   - Need comprehensive monitoring
   - Alert on anomalies

3. **No Backups**
   - Data loss incidents due to no backups
   - Regular backups are critical
   - Test restore procedures

4. **Poor Capacity Planning**
   - Traffic spikes cause outages
   - Load test before events
   - Auto-scaling is essential

5. **Security Neglect**
   - Many breaches due to poor security
   - Keep software updated
   - Security monitoring

### Best Practices

1. **Safeguards**
   - Require confirmation for destructive operations
   - Rate limiting for dangerous operations
   - Dry-run mode for testing

2. **Monitoring**
   - Comprehensive monitoring
   - Alert on anomalies
   - Dashboard for key metrics

3. **Backups**
   - Regular automated backups
   - Multiple backup locations
   - Test restore procedures

4. **Testing**
   - Load testing
   - Chaos engineering
   - Regular failover testing

5. **Documentation**
   - Runbooks for common operations
   - Incident response procedures
   - Architecture documentation

6. **Security**
   - Regular security updates
   - Security monitoring
   - Access controls
   - Secret management

---

## Interview Questions

### Q1: Describe a production incident you handled.

**Answer Structure:**
1. **Context**: What was the system?
2. **Incident**: What happened?
3. **Impact**: Who was affected?
4. **Root Cause**: What caused it?
5. **Solution**: How did you fix it?
6. **Prevention**: How to prevent it?

**Example:**
"I was on-call when our payment service started failing. Users couldn't complete payments. I investigated and found the database connection pool was exhausted. I increased the pool size and added monitoring. Long-term, I implemented connection pooling best practices and added alerts."

---

### Q2: How do you prevent production incidents?

**Answer:**
1. **Safeguards**: Require confirmation for destructive operations
2. **Monitoring**: Comprehensive monitoring and alerting
3. **Testing**: Load testing, chaos engineering
4. **Backups**: Regular backups, test restore
5. **Documentation**: Runbooks, procedures
6. **Security**: Regular updates, security monitoring
7. **Code Review**: Peer review for critical changes
8. **Staging**: Test in staging before production

---

### Q3: How do you handle a production incident?

**Answer:**
1. **Assess**: Understand the impact
2. **Communicate**: Notify stakeholders
3. **Investigate**: Find root cause
4. **Mitigate**: Stop the bleeding
5. **Fix**: Resolve the issue
6. **Verify**: Ensure fix works
7. **Post-mortem**: Document and learn
8. **Prevent**: Implement prevention measures

---

## Summary

Production incidents teach valuable lessons:

✅ **Human Error**: Design for mistakes
✅ **Monitoring**: Critical for detection
✅ **Backups**: Essential for recovery
✅ **Testing**: Prevent issues before production
✅ **Security**: Never neglect
✅ **Documentation**: Helps during incidents
✅ **Communication**: Keep stakeholders informed

**Key Takeaways:**
- Incidents will happen - design for them
- Learn from every incident
- Share knowledge with team
- Implement prevention measures
- Regular testing and drills

---

*This guide is continuously updated with new incidents and lessons learned.*

