Ce que fait ce playbook :
<ul>
<li>Vérification de la version de PVE :</li>
<pre>
 - name: version PVE
   shell: pveversion
   register: release
</pre>
<li>Affichage de la version de PVE :</li>
<pre>
- name: Notification version PVE
  debug: msg="Version de PVE {{ release.stdout }}"
</pre>
<li>Message d'activation du mode maintenance du noeud :</li>
<pre>
- name: mode maintenance
  debug: msg="Mode maintenance activé"
 </pre>
<li>Activation du mode maintenance du noeud :</li>
 <pre>
- name: maintenance on
  shell: ha-manager crm-command node-maintenance enable $(hostname) 
 </pre>
 <li>Tempo de 5 mintes</li>li>
 <pre>
  - name: Attendre 5 minutes pour la fin de migration des VMs
      ansible.builtin.wait_for:
        timeout: 300
      delegate_to: localhost
 </pre>
<li>Mise à jour des dépôts :</li>
<pre>
- name: Mise à jour des dépôts
  apt: update_cache=yes
</pre>
<li>Mise à jour des paquets :</li>
<pre>
- name: Mise à jour des paquets
  apt: upgrade=dist
</pre>
<li>Vérification de la dernière version du noyau Linux installé sur le Proxmox :</li>
<pre>
- name: version Kernel dispo
  shell: ls -t /boot/vmlinuz-* | sed "s/\/boot\/vmlinuz-//g" | head -n1
  register: kernel_dispo
</pre>
 <li>Vérification de la version du noyau Linux utilisé par Proxmox :</li>
<pre>
- name: version kernel actuel
  shell: uname -r
  register: kernel_actuel
</pre>
<li>Comparaison entre les versions du noyau Linux disponible et celle utilisée par Proxmox. Si la version utilisée est ancienne, message d’avertissement pour redémarrer le Proxmox :</li>
<pre>
 - name: vérification version kernel
   debug: msg="Ce PVE doit être redémarré, kernel actuel {{ kernel_actuel.stdout }} kernel disponible {{ kernel_dispo.stdout }}"
   when: kernel_dispo.stdout != kernel_actuel.stdout
</pre>
<li>Revérification de la version de PVE :</li>
<pre>
- name: Vérification de la version de PVE
  shell:  pveversion
  register: new_release
</pre>
<li>Notification de la mise à niveau de la version de PVE si c'st le cas :</li>
<pre>
- name: Notification de la mise à niveau de la version de PVE
  debug: msg="PVE à changé de version {{ release.stdout }} à {{ new_release.stdout }}"
  when: release.stdout != new_release.stdout
</pre>
<li>Vérification de la présence du paquet needrestart, dans le cas échéant l'installer :</li>
<pre>
- name: Vérification de la présence de needrestart
  apt: name=needrestart state=present
 </pre>
<li>Lister les services à redémarrer :</li>
<pre>
- name: Lister les services a rédémarrer
  shell: needrestart -rl
  register: services
</pre>
<li>Affichage des services à redémarrer :</li>
<pre>
- name: Afficher les services à redémarrer
  debug: msg="{{ services.stdout_lines }}"
</pre>
<li>Redémarrer les services :</li>
<pre>
- name: Redémarrage des services
  shell: needrestart -ra
</pre>
<li>Message de désactivation du le mode maintenance :</li>
 <pre>
- name: mode maintenance
  debug: msg="Mode maintenance dédactivé"
  </pre>
<li>Désactivation du le mode maintenance :</li>
  <pre>
- name: maintenance off
  shell: ha-manager crm-command node-maintenance disable $(hostname)
   </pre>
</ul>
