# Auditing SELinux Events

For SELinux there are two main types of audit event:

1.  **AVC Audit Events** - These are generated by the AVC subsystem as a
    result of access denials, or where specific events have requested an
    audit message (i.e. where an `auditallow` rule has been used in
    the policy).
2.  **SELinux-aware Application Events** - These are generated by the
    SELinux kernel services and SELinux-aware applications for events
    such as system errors, initialisation, policy load, changing boolean
    states, setting of enforcing / permissive mode, relabeling etc.

The audit and event messages are generally stored in one of the
following logs (in F-27 anyway):

1.  The SELinux kernel boot events are logged in the */var/log/dmesg* log.
2.  The system log */var/log/messages* contains messages generated by
    SELinux before the audit daemon has been loaded.
3.  The audit log */var/log/audit/audit.log* contains events that take
    place after the audit daemon has been loaded. The AVC audit messages
    of interest are described in the [AVC Audit Events](#avc-audit-events)
    section with others described in the
    [General SELinux Audit Events](#general-selinux-audit-events)
    section. Fedora uses the audit framework **auditd**(8) as standard.

Notes:

1.  It is not mandatory for SELinux-aware applications to audit events
    or even log them in the audit log. The decision is made by the
    application designer.
2.  The format of audit messages do not need to conform to any format,
    however where possible applications should use the
    ***audit_log_user_avc_message**(3)* function with a suitably
    formatted message if using ***auditd**(8)*. The type of audit events
    possible are defined in the *include/libaudit.h* and
    *include/linux/audit.h* files.
3.  Those libselinux library functions that output messages do so to
    `stderr` by default, however this can be changed by calling
    ***selinux_set_callback**(3)* and specifying an alternative log
    handler.

<br>

## AVC Audit Events

**Table 1** describes the general format of AVC audit
messages in the audit.log when access has been denied or an audit event
has been specifically requested. Other types of events are shown in the
section that follows.

<table>
<tbody>
<tr style="background-color:#D3D3D3;">
<td><strong>Keyword<strong></td>
<td><strong>Description<strong></td>
</tr>
<tr>
<td>type</td>
<td><p>For SELinux AVC events this can be:</p>
<p>type=AVC - for kernel events</p>
<p>type=USER_AVC - for user-space object manager events</p>
<p>Note that once the AVC event has been logged, another event with type=SYSCALL may follow that contains further information regarding the event. </p>
<p>The AVC event can always be tied to the relevant SYSCALL event as they have the same serial_number in the msg=audit(time:serial_number) field as shown in the following example:</p>
<p><strong>type=AVC</strong> msg=audit(1243332701.744<strong>:101</strong>): avc: denied { getattr } for pid=2714 comm="ls" path="/usr/lib/locale/locale-archive" dev=dm-0 ino=353593 scontext=system_u:object_r:unlabeled_t:s0 tcontext=system_u:object_r:locale_t:s0 tclass=file</p>
<p><strong>type=SYSCALL</strong> msg=audit(1243332701.744<strong>:101</strong>): arch=40000003 syscall=197 success=yes exit=0 a0=3 a1=553ac0 a2=552ff4 a3=bfc5eab0 items=0 ppid=2671 pid=2714 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts1 ses=1 comm="ls" <em>exe="/bin/ls</em>" subj=system_u:object_r:unlabeled_t:s0 key=(null)</p></td>
</tr>
<tr>
<td>msg</td>
<td>This will contain the audit keyword with a reference number (e.g. msg=audit(1243332701.744:101))</td>
</tr>
<tr>
<td>avc</td>
<td><p>This will be either denied when access has been denied or granted when an <em><em>auditallow</em><em> rule</em></em> has been defined by the policy.</p>
<p>The entries that follow the `avc=` field depend on what type of event is being audited. Those shown below are generated by the kernel AVC audit function, however the user space AVC audit function will return fields relevant to the application being managed by their Object Manager.</p></td>
</tr>
<tr>
<td>pid</td>
<td rowspan="2">If a task, then log the process id (pid) and the name of the executable file (comm).</td>
</tr>
<tr>
<td>comm</td>
</tr>
<tr>
<td>capability</td>
<td>If a capability event then log the identifier.</td>
</tr>
<tr>
<td>path</td>
<td rowspan="4">If a File System event then log the relevant information. Note that the name field may not always be present.</td>
</tr>
<tr>
<td>name</td>
</tr>
<tr>
<td>dev</td>
</tr>
<tr>
<td>ino</td>
</tr>
<tr>
<td>laddr</td>
<td rowspan="4">If a Socket event then log the Source / Destination addresses and ports for IP4 or IP6 sockets (AF_INET).</td>
</tr>
<tr>
<td>lport</td>
</tr>
<tr>
<td>faddr</td>
</tr>
<tr>
<td>fport</td>
</tr>
<tr>
<td>path</td>
<td>If a File Socket event then log the path (AF_UNIX).</td>
</tr>
<tr>
<td>saddr</td>
<td rowspan="5"><p>If a Network event then log the Source / Destination addresses and ports with the network interface for IP4 or IP6 networks (AF_INET).</p></td>
</tr>
<tr>
<td>src</td>
</tr>
<tr>
<td>daddr</td>

</tr>
<tr>
<td>dest</td>
</tr>
<tr>
<td>netif</td>
</tr>
<tr>
<td>sauid</td>
<td rowspan="3">IPSec security association identifiers</td>
</tr>
<tr>
<td>hostname</td>
</tr>
<tr>
<td>addr</td>
</tr>
<tr>
<td>resid</td>
<td rowspan="2">X-Windows resource ID and type.</td>
</tr>
<tr>
<td>restype</td>
</tr>
<tr>
<td>scontext</td>
<td>The security context of the source or subject.</td>
</tr>
<tr>
<td>tcontext</td>
<td>The security context of the target or object.</td>
</tr>
<tr>
<td>tclass</td>
<td>The object class of the target or object.</td>
</tr>
</tbody>
</table>

**Table 1: AVC Audit Message Description**

Example *audit.log* denied and granted events are shown in the following
examples:

```
# This is an example **denied** message - note that there are two
# type=AVC calls, but only one corresponding type=SYSCALL entry.
type=AVC msg=audit(1242575005.122:101): avc: denied { rename } for
pid=2508 comm="canberra-gtk-pl"
name="c73a516004b572d8c845c74c49b2511d:runtime.tmp" dev=dm-0 ino=188999
scontext=test_u:staff_r:oddjob_mkhomedir_t:s0
tcontext=test_u:object_r:gnome_home_t:s0 tclass=lnk_file

type=AVC msg=audit(1242575005.122:101): avc: denied { unlink } for
pid=2508 comm="canberra-gtk-pl"
name="c73a516004b572d8c845c74c49b2511d:runtime" dev=dm-0 ino=188578
scontext=test_u:staff_r:oddjob_mkhomedir_t:s0
tcontext=system_u:object_r:gnome_home_t:s0 tclass=lnk_file

type=SYSCALL msg=audit(1242575005.122:101): arch=40000003 syscall=38
success=yes exit=0 a0=82d2760 a1=82d2850 a2=da6660 a3=82cb550 items=0
ppid=2179 pid=2508 auid=500 uid=500 gid=500 euid=500 suid=500 fsuid=500
egid=500 sgid=500 fsgid=500 tty=(none) ses=1 comm="canberra-gtk-pl"
exe="/usr/bin/canberra-gtk-play"
subj=test_u:staff_r:oddjob_mkhomedir_t:s0 key=(null)
```

```
# These are example X-Windows object manager audit message:
type=USER_AVC msg=audit(1267534171.023:18): user pid=1169 uid=0
auid=4294967295 ses=4294967295
subj=system_u:unconfined_r:unconfined_t msg='avc: denied { getfocus }
for request=X11:GetInputFocus comm=X-setest xdevice="Virtual core
keyboard" scontext=unconfined_u:unconfined_r:x_select_paste_t
tcontext=system_u:unconfined_r:unconfined_t tclass=x_keyboard :
exe="/usr/bin/Xorg" sauid=0 hostname=? addr=? terminal=?'

type=USER_AVC msg=audit(1267534395.930:19): user pid=1169 uid=0
auid=4294967295 ses=4294967295
subj=system_u:unconfined_r:unconfined_t msg='avc: denied { read } for
request=SELinux:SELinuxGetClientContext comm=X-setest resid=3c00001
restype=&lt;unknown&gt;
scontext=unconfined_u:unconfined_r:x_select_paste_t
tcontext=unconfined_u:unconfined_r:unconfined_t tclass=x_resource :
exe="/usr/bin/Xorg" sauid=0 hostname=? addr=? terminal=?'
```

```
# This is an example **granted** audit message:
type=AVC msg=audit(1239116352.727:311): avc: granted { transition } for
pid=7687 comm="bash" path="/usr/move_file/move_file_c" dev=dm-0
ino=402139 scontext=unconfined_u:unconfined_r:unconfined_t
tcontext=unconfined_u:unconfined_r:move_file_t tclass=process

type=SYSCALL msg=audit(1239116352.727:311): arch=40000003 syscall=11
success=yes exit=0 a0=8a6ea98 a1=8a56fa8 a2=8a578e8 a3=0 items=0
ppid=2660 pid=7687 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0
sgid=0 fsgid=0 tty=(none) ses=1 comm="move_file_c"
exe="/usr/move_file/move_file_c"
subj=unconfined_u:unconfined_r:move_file_t key=(null)
```

<br>

## General SELinux Audit Events

This section shows a selection of non-AVC SELinux-aware services audit
events taken from the audit.log. For a list of valid *type=* entries,
the following include files should be consulted: *include/libaudit.h*
and *include/linux/audit.h*.

Note that there can be what appears to be multiple events being
generated for the same event. For example the kernel security server
will generate a `MAC_POLICY_LOAD` event to indicate that the policy
has been reloaded, but then each userspace object manager could then
generate a `USER_MAC_POLICY_LOAD` event to indicate that it had also
processed the event.

Policy reload - `MAC_POLICY_LOAD`, `USER_MAC_POLICY_LOAD` - These
events were generated when the policy was reloaded.

```
type=MAC_POLICY_LOAD msg=audit(1336662937.117:394): policy loaded
auid=0 ses=2

type=SYSCALL msg=audit(1336662937.117:394): arch=c000003e syscall=1
success=yes exit=4345108 a0=4 a1=7f0a0c547000 a2=424d14 a3=7fffe3450f20
items=0 ppid=3845 pid=3848 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0
egid=0 sgid=0 fsgid=0 tty=pts2 ses=2 comm="load_policy"
exe="/sbin/load_policy"
subj=unconfined_u:unconfined_r:load_policy_t:s0-s0:c0.c1023
key=(null)

type=USER_MAC_POLICY_LOAD msg=audit(1336662938.535:395): pid=0 uid=0
auid=4294967295 ses=4294967295
subj=system_u:system_r:xserver_t:s0-s0:c0.c1023 msg='avc: received
policyload notice (seqno=2) : exe="/usr/bin/Xorg" sauid=0 hostname=?
addr=? terminal=?'
```

<br>

Change enforcement mode - `MAC_STATUS` - This was generated when the
SELinux enforcement mode was changed:

```
type=MAC_STATUS msg=audit(1336836093.835:406): enforcing=1
old_enforcing=0 auid=0 ses=2

type=SYSCALL msg=audit(1336836093.835:406): arch=c000003e syscall=1
success=yes exit=1 a0=3 a1=7fffe743f9e0 a2=1 a3=0 items=0 ppid=2047
pid=5591 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0
tty=pts0 ses=2 comm="setenforce" exe="/usr/sbin/setenforce"
subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
```

<br>

Change boolean value - `MAC_CONFIG_CHANGE` - This event was generated
when ***setsebool**(8)* was run to change a boolean. Note that the
bolean name plus new and old values are shown in the
`MAC_CONFIG_CHANGE` type event with the `SYSCALL` event showing what
process executed the change.

```
type=MAC_CONFIG_CHANGE msg=audit(1336665376.629:423):
bool=domain_paste_after_confirm_allowed val=0 old_val=1 auid=0
ses=2

type=SYSCALL msg=audit(1336665376.629:423): arch=c000003e syscall=1
success=yes exit=2 a0=3 a1=7fff42803200 a2=2 a3=7fff42803f80 items=0
ppid=2015 pid=4664 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0
sgid=0 fsgid=0 tty=pts0 ses=2 comm="setsebool" exe="/usr/sbin/setsebool"
subj=unconfined_u:unconfined_r:setsebool_t:s0-s0:c0.c1023 key=(null)

NetLabel - *MAC_UNLBL_STCADD* - Generated when adding a static
non-mapped label. There are many other NetLabel events possible, such
as: *MAC_MAP_DEL*, *MAC_CIPSOV4_DEL* *...*

type=MAC_UNLBL_STCADD msg=audit(1336664587.640:413): netlabel: auid=0
ses=2 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
netif=lo src=127.0.0.1
sec_obj=system_u:object_r:unconfined_t:s0-s0:c0,c100 res=1

type=SYSCALL msg=audit(1336664587.640:413): arch=c000003e syscall=46
success=yes exit=96 a0=3 a1=7fffde77f160 a2=0 a3=666e6f636e753a72
items=0 ppid=2015 pid=4316 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0
egid=0 sgid=0 fsgid=0 tty=pts0 ses=2 comm="netlabelctl"
exe="/sbin/netlabelctl"
subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
```

<br>

Labeled IPSec - `MAC_IPSEC_EVENT` - Generated when running
***setkey**(8)* to load IPSec configuration:

```
type=MAC_IPSEC_EVENT msg=audit(1336664781.473:414): op=SAD-add auid=0
ses=2 subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
sec_alg=1 sec_doi=1
sec_obj=system_u:system_r:postgresql_t:s0-s0:c0,c200 src=127.0.0.1
dst=127.0.0.1 spi=592(0x250) res=1

type=SYSCALL msg=audit(1336664781.473:414): arch=c000003e syscall=44
success=yes exit=176 a0=4 a1=7fff079d5100 a2=b0 a3=0 items=0 ppid=2015
pid=4356 auid=0 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0
tty=pts0 ses=2 comm="setkey" exe="/sbin/setkey"
subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)

SELinux kernel errors - *SELINUX_ERR* - These example events were
generated by the kernel security server. These were generated by the
kernel security server because *anon_webapp_t* has been give
privileges that are greater than that given to the process that started
the new thread (this is not allowed).

type=SELINUX_ERR msg=audit(1311948547.151:138):
op=security_compute_av reason=bounds
scontext=system_u:system_r:anon_webapp_t:s0-s0:c0,c100,c200
tcontext=system_u:object_r:security_t:s0 tclass=dir
perms=ioctl,read,lock

type=SELINUX_ERR msg=audit(1311948547.151:138):
op=security_compute_av reason=bounds
scontext=system_u:system_r:anon_webapp_t:s0-s0:c0,c100,c200
tcontext=system_u:object_r:security_t:s0 tclass=file
perms=ioctl,read,write,getattr,lock,append,open

These were generated by the kernel security server when an SELinux-aware
application was trying to use ***setcon***(3) to create a new thread. To
fix this a *typebounds* statement is required in the policy.

type=SELINUX_ERR msg=audit(1311947138.440:126):
op=security_bounded_transition result=denied
oldcontext=system_u:system_r:httpd_t:s0-s0:c0.c300
newcontext=system_u:system_r:anon_webapp_t:s0-s0:c0,c100,c200

type=SYSCALL msg=audit(1311947138.440:126): arch=c000003e syscall=1
success=no exit=-1 a0=b a1=7f1954000a10 a2=33 a3=6e65727275632f72
items=0 ppid=3295 pid=3473 auid=4294967295 uid=48 gid=48 euid=48 suid=48
fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4294967295 comm="httpd"
exe="/usr/sbin/httpd" subj=system_u:system_r:httpd_t:s0-s0:c0.c300
key=(null)
```

<br>

Role changes - `USER_ROLE_CHANGE` - Used ***newrole**(1)* to set a new
role that was not valid.

```
type=USER_ROLE_CHANGE msg=audit(1336837198.928:429): pid=0 uid=0
auid=0 ses=2
subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
msg='newrole:
old-context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
new-context=?: exe="/usr/bin/newrole" hostname=? addr=?
terminal=/dev/pts/0 res=failed'
```

<br>

<!-- %CUTHERE% -->

---
**[[ PREV ]](modes.md)** **[[ TOP ]](#)** **[[ NEXT ]](polyinstantiation.md)**