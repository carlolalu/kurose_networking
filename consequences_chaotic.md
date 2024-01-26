We investigate on what it is that I must do in this project. How do I completely disable smb? I suspect from the code of `generate`. Anyway the first thing is likely to be grepping for the example `dhcp_enable`:


```
saturn@carlo$:[staging]> grep -r DHCP_ENABLE .
./installation/defaultvalues-intranator.utf8.cnf:122 DHCP_ENABLE,0: "1"
./libcnfcheck/src/check_var/cnfcheck_network.cpp:    cv=new CHECK_VAR("DHCP_ENABLE",true, "enable the DHCP server");
./libcnfcheck/src/check_var/cnfcheck_network.cpp:                        .then_(h::If(h::Jump().search("DHCP_ENABLE",  new STRING_EQ_CHECK("0")))
./libcnfcheck/test_check_var/network.cpp:    int ok1 = cf->add_cnf(CNF_LINE("DHCP_ENABLE",0,"1"));
./libcnfcheck/test_check_var/network.cpp:    cf->add_cnf(CNF_LINE("DHCP_ENABLE",0,"1"));
./libcnfcheck/test_check_var/network.cpp:    cf->add_cnf(CNF_LINE("DHCP_ENABLE",0,"1"));
./libcnfcheck/test_check_var/network.cpp:    cf->add_cnf(CNF_LINE("DHCP_ENABLE",0,"0"));
./libcnfcheck/test_check_var/network.cpp:    cf->add_cnf(CNF_LINE("DHCP_ENABLE",0,"0"));
./dhcpd_warden/dhcpd_warden.init:        if [ "$DHCP_ENABLE" == "1" ]; then
./dhcpd_warden/dhcpd_warden.init:        if [ "$DHCP_ENABLE" == "1" ]; then
./dhcpd_warden/dhcpd_warden.init:            if [ "$DHCP_ENABLE" == "1" ]; then
./ui/cmake-build-debug-intradev-carlo/ui/name2page.cpp:    name2page["DHCP_ENABLE"] = "network_intranet_dhcp";
grep: ./ui/build/ui/libui.so: binary file matches
grep: ./ui/build/ui/CMakeFiles/ui.dir/name2page.cpp.o: binary file matches
grep: ./ui/build/ui/CMakeFiles/ui.dir/form_product_information.cpp.o: binary file matches
grep: ./ui/build/ui/CMakeFiles/ui.dir/form_netzwerk_dhcp.cpp.o: binary file matches
./ui/build/ui/name2page.cpp:    name2page["DHCP_ENABLE"] = "network_intranet_dhcp";
./ui/ui/form_netzwerk_dhcp.cpp:    unique = "DHCP_ENABLE";
./ui/ui/form_netzwerk_dhcp.cpp:    addArniedVar ("DHCP_ENABLE");
./ui/ui/form_netzwerk_dhcp.cpp:    showCheckBox("DHCP_ENABLE");
./ui/ui/form_netzwerk_dhcp.cpp:    getCheckBox ("DHCP_ENABLE");
./ui/ui/form_netzwerk_dhcp.cpp:    arnied_vars["DHCP_ENABLE"] = "1";
./ui/ui/form_product_information.cpp:  changed_vars.push_back(CNF_VAR("DHCP_ENABLE", 0, "1"));
./generate/generate/mon.ag:    if (getcnf(ac, "DHCP_ENABLE", 0) == "1" && count_enabled_dhcp_nics(ac) > 0)
./generate/generate/crontab.ag:    if (getcnf(ac, "DHCP_ENABLE", 0) == "1" && count_enabled_dhcp_nics(ac) > 0)
./generate/generate/dhcpd_define.ag:    outf << "DHCP_ENABLE=" << getcnf(ac, "DHCP_ENABLE", 0) << "\n";
./ui-data/preprocess/input/network_intranet_dhcp.html:<name>DHCP_ENABLE</name>
./arniesetup/arniesetup/main_tvision.cpp:            CNF_VAR var("DHCP_ENABLE", 0, "0");
./arniesetup/arniesetup/main_tvision.cpp:    if (data_get_cnf("DHCP_ENABLE", 0) == "1")
./arniesetup/arniesetup/main_tvision.cpp:            data_set_cnf ("DHCP_ENABLE", 0, "1");
./arniesetup/arniesetup/main_tvision.cpp:            data_set_cnf ("DHCP_ENABLE", 0, "0");
```


notice also `./generate/generate/mon.ag:` which tells us what is a mon check!!!

btw, by grepping for ad-related cnf_vars you can come out with such a thing:
```
intradev-carlo@root#[~]:> get_cnf | grep AD
3    ADTASKS_CRON,0: "0123456"
4       (3) ADTASKS_CRON_BEGIN,0: "0"
5       (3) ADTASKS_CRON_END,0: "86400"
6       (3) ADTASKS_CRON_EVERY,0: "3600"
7    AD_ENABLE_UI,0: "0"
8    AD_LDAP_BIND_ACCOUNT,0: ""
9    AD_LDAP_BIND_PASSWORD,0: ""
10   AD_MACHINE,0: ""
11   AD_MACHINE_CHANGEPW_INTERVAL,0: "30"
12   AD_MACHINE_CREDENTIALS,0: ""
```


I found another useful place in which dhcp_enable is odcumented in generate:
```
// DHCPd is special: It doesn't start if another DHCP server is running
    // in the same network. We take care of this in the restart script
    // though an error will still be logged.
    if (getcnf(ac, "DHCP_ENABLE", 0) == "1" && count_enabled_dhcp_nics(ac) > 0)
```


Besides, in the same file, there is the `nmbd script` nominated, and right before the AD_MACHINE_CREDENTIALS!
```
    outf << "    service samba-smbd\n";
    outf << "        interval 5m\n";
    outf << "        monitor program.monitor smbd ;;\n";
    outf << "        period wd {Mon-Sun}\n";
    outf << "            alertafter 2 6m\n";
    outf << "            alert restart_smb.alert\n";
    outf << "    service samba-nmbd\n";
    outf << "        interval 5m\n";
    outf << "        monitor program.monitor nmbd ;;\n";
    outf << "        period wd {Mon-Sun}\n";
    outf << "            alertafter 2 6m\n";
    outf << "            alert restart_nmb.alert\n";

    if (getcnf (ac, "AD_MACHINE_CREDENTIALS", 0) != "")
```


