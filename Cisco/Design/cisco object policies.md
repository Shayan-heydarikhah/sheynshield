object group + acl : on routers :
 r1 :
object-group services myservice tcp 443 tcp 80 tcp 3389 udp 53
object-group network lan 192.168.100.0 /24 
object-group network srv 172.16.15.0 /24 
ip access-list extended fw permit object-group myservice object-group lan object-group srv
