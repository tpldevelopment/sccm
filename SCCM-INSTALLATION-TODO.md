# SCCM Installation Todo List

## Pre-Installation Planning

- [ ] Review hardware and software requirements
  - [ ] Verify server meets minimum requirements (4-core CPU, 16GB RAM minimum)
  - [ ] Ensure adequate disk space (150GB+ recommended)
  - [ ] Check Windows Server version (2019/2022 recommended)
- [ ] Obtain necessary licenses
  - [ ] Windows Server licenses
  - [ ] SQL Server licenses
  - [ ] SCCM/ConfigMgr licenses
- [ ] Plan network topology and site hierarchy
  - [ ] Determine primary site vs CAS (Central Administration Site)
  - [ ] Plan distribution points locations
  - [ ] Define site boundaries and boundary groups
- [ ] Document installation parameters
  - [ ] Site code (3 characters)
  - [ ] Site name
  - [ ] Installation path
  - [ ] SQL Server instance name

## Active Directory Preparation

- [ ] Extend Active Directory schema for SCCM
  - [ ] Run `extadsch.exe` from SCCM installation media
  - [ ] Verify schema extension in `ExtADSch.log`
- [ ] Create System Management container in AD
  - [ ] Grant full control permissions to site server computer account
- [ ] Configure Active Directory System Discovery
- [ ] Set up service accounts
  - [ ] SQL Server service account
  - [ ] SCCM site server service account
  - [ ] Network access account
  - [ ] Client push installation account

## SQL Server Installation

- [ ] Install SQL Server (2016/2019/2022)
  - [ ] Enable SQL Server Agent
  - [ ] Configure SQL Server memory settings (leave 4GB for OS)
  - [ ] Set SQL Server collation to `SQL_Latin1_General_CP1_CI_AS`
- [ ] Install SQL Server Reporting Services (SSRS)
  - [ ] Configure SSRS for native mode
  - [ ] Verify SSRS is running
- [ ] Configure SQL Server network protocols
  - [ ] Enable TCP/IP protocol
  - [ ] Configure firewall rules (port 1433)
- [ ] Create SCCM database (will be done by installer, verify permissions)

### Optional: Configure SQL Server Always On Availability Groups (HA)

- [ ] **Prerequisites for Always On**
  - [ ] Minimum 2 SQL Server nodes (3 recommended for automatic failover)
  - [ ] Windows Server Failover Clustering (WSFC) installed on all nodes
  - [ ] All nodes in same Active Directory domain
  - [ ] SQL Server Enterprise Edition on all nodes
  - [ ] Shared storage or Storage Spaces Direct (not required for AG, only for WSFC quorum)
- [ ] **Install and configure Windows Failover Clustering**
  - [ ] Install Failover Clustering feature on all SQL nodes
  - [ ] Run cluster validation wizard
  - [ ] Create Windows failover cluster
  - [ ] Configure cluster quorum (File Share Witness or Cloud Witness recommended)
  - [ ] Add all SQL nodes to the cluster
- [ ] **Configure SQL Server for Always On**
  - [ ] Enable Always On Availability Groups in SQL Server Configuration Manager on each node
  - [ ] Restart SQL Server service on each node
  - [ ] Verify SQL Server is running under domain account (required for AG)
- [ ] **Create Availability Group**
  - [ ] Create availability group listener name (virtual network name)
  - [ ] Configure listener IP address (static IP in same subnet)
  - [ ] Add primary replica (initial SQL node)
  - [ ] Add secondary replica(s) with synchronous commit mode
  - [ ] Configure automatic failover for synchronous replicas
  - [ ] Set readable secondary (optional - for read-only reporting)
- [ ] **Configure SCCM database for Always On**
  - [ ] Install SCCM site database on primary replica
  - [ ] Add SCCM database to availability group
  - [ ] Wait for database synchronization to complete
  - [ ] Configure SCCM to use availability group listener name (not direct server name)
  - [ ] Verify SCCM site database is in "Synchronized" state
- [ ] **Test failover**
  - [ ] Perform manual failover to secondary replica
  - [ ] Verify SCCM site remains operational
  - [ ] Test automatic failover (stop SQL service on primary)
  - [ ] Verify SCCM reconnects to new primary
- [ ] **Configure monitoring and alerts**
  - [ ] Set up SQL Server Agent alerts for AG health
  - [ ] Configure email notifications for failover events
  - [ ] Monitor synchronization lag and health dashboard

## Windows Server Configuration

- [ ] Install required Windows Server roles and features
  - [ ] .NET Framework 3.5
  - [ ] .NET Framework 4.8 or later
  - [ ] Windows ADK (Assessment and Deployment Kit)
  - [ ] Windows PE add-on for ADK
  - [ ] BITS (Background Intelligent Transfer Service)
  - [ ] Remote Differential Compression
- [ ] Configure Windows Firewall
  - [ ] Allow SCCM required ports (80, 443, 445, 1433, 10123, etc.)
  - [ ] Create inbound rules for client communication
- [ ] Install IIS (Internet Information Services)
  - [ ] Enable required IIS features
  - [ ] Configure WSUS if using software update point
- [ ] Configure system environment
  - [ ] Set proper time zone
  - [ ] Verify DNS configuration
  - [ ] Test network connectivity

## SCCM Installation

- [ ] Download latest SCCM current branch version
- [ ] Run prerequisite checker
  - [ ] Review and resolve any warnings/errors
  - [ ] Document prerequisite check results
- [ ] Launch SCCM setup wizard
  - [ ] Select installation type (Primary Site or CAS)
  - [ ] Enter product key
  - [ ] Accept license terms
- [ ] Configure site settings
  - [ ] Enter site code and site name
  - [ ] Specify installation directory
  - [ ] Configure SQL Server settings
- [ ] Select site system roles to install
  - [ ] Management Point
  - [ ] Distribution Point
  - [ ] Software Update Point (if needed)
  - [ ] Reporting Services Point
- [ ] Configure customer experience improvement program participation
- [ ] Begin installation and monitor progress
  - [ ] Monitor installation logs in `C:\ConfigMgrSetup.log`
  - [ ] Verify no critical errors

## Post-Installation Configuration

- [ ] Verify installation success
  - [ ] Check site status in SCCM console
  - [ ] Review component status
  - [ ] Verify all site system roles are functioning
- [ ] Install SCCM console on admin workstation
- [ ] Configure discovery methods
  - [ ] Active Directory System Discovery
  - [ ] Active Directory User Discovery
  - [ ] Active Directory Group Discovery
  - [ ] Network Discovery (if needed)
- [ ] Configure boundaries and boundary groups
  - [ ] Create IP subnet boundaries
  - [ ] Associate distribution points with boundary groups
  - [ ] Configure boundary group relationships
- [ ] Set up client push installation
  - [ ] Configure client push installation accounts
  - [ ] Enable automatic site-wide client push
  - [ ] Configure client push installation properties
- [ ] Configure software update point
  - [ ] Synchronize WSUS
  - [ ] Select product categories and classifications
  - [ ] Configure synchronization schedule
- [ ] Configure reporting services point
  - [ ] Verify report server URL
  - [ ] Test report access
- [ ] Set up email notifications (optional)
  - [ ] Configure SMTP settings
  - [ ] Test alert notifications

## Security and Maintenance

- [ ] Configure role-based administration (RBA)
  - [ ] Create security roles
  - [ ] Assign administrative users
  - [ ] Configure security scopes
- [ ] Configure backup maintenance task
  - [ ] Enable site server backup
  - [ ] Configure backup schedule and location
  - [ ] Test backup restoration procedure
- [ ] Set up status message queries and alerts
- [ ] Configure log file retention policies
- [ ] Document installation and configuration
  - [ ] Record all passwords and service accounts
  - [ ] Document custom configurations
  - [ ] Create disaster recovery plan

## Testing and Validation

- [ ] Deploy test client to workstation
  - [ ] Verify client installation
  - [ ] Check client communication with management point
  - [ ] Confirm client appears in console
- [ ] Test application deployment
  - [ ] Create test application package
  - [ ] Deploy to test collection
  - [ ] Verify successful installation
- [ ] Test software updates
  - [ ] Create test update deployment
  - [ ] Verify updates install correctly
- [ ] Test reporting functionality
  - [ ] Run sample reports
  - [ ] Create custom report
- [ ] Perform end-to-end testing of all major features

## Production Rollout

- [ ] Create production collections
  - [ ] Windows 10/11 collections
  - [ ] Server collections
  - [ ] Departmental collections
- [ ] Begin gradual client deployment
  - [ ] Pilot group (IT department)
  - [ ] Phase 2 deployment
  - [ ] Full production rollout
- [ ] Monitor deployment progress
  - [ ] Check client installation success rate
  - [ ] Review error logs and troubleshoot issues
- [ ] Train administrative staff
- [ ] Create operational runbooks and procedures

## Ongoing Maintenance

- [ ] Schedule regular SCCM updates
- [ ] Monitor site and component health daily
- [ ] Review and clean up old packages/applications
- [ ] Optimize database maintenance
- [ ] Review and update collections membership
- [ ] Conduct regular backup verifications

---

**Notes:**
- Installation typically takes 4-8 hours depending on environment
- Always test in a lab environment first if possible
- Keep detailed documentation of all configurations
- Plan for rollback procedures

**Estimated Timeline:** 3-5 days for full installation and initial configuration
