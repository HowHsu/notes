```c


                                           +-------------------------+      +-------------------------+   +-------------------------+
  +--------------------------------------->|     struct mount        |      |     struct mount        |   |     struct mount        |
  |                                        +-------------------------+      +-------------------------+   +-------------------------+
  |                                        | +---------------------+ |      | +---------------------+ |   | +---------------------+ |
  |                                        | | struct vfsmount mnt | |      | | struct vfsmount mnt | |   | | struct vfsmount mnt | |
  |                                        | +---------------------+ |      | +---------------------+ |   | +---------------------+ |
  |                                        | |       mnt_root      |-|--+   | |       mnt_root      | |   | |       mnt_root      | |
  |                  +---------------------|-|        mnt_sb       | |  |   | |        mnt_sb       | |   | |        mnt_sb       | |
  |                  |                     | +---------------------+ |  |   | +---------------------+ |   | +---------------------+ |
  |                  |                     |                         |  |   |                         |   |                         |
  |                  |             +------>|      mnt_instance       |----->|      mnt_instance       |-->|      mnt_instance       |
  |                  |             |       |                         |  |   |                         |   |                         |
  |                  |             |       +-------------------------+  |   +-------------------------+   +-------------------------+
  |                  |             |                                    |
  |                  |             |                                    |
  |                  |             |                                    |
  |                  |             |                                    |
  |                  V             |                                    |
  |    +----------------------+    |                                    |
  |    |       superblock     |    |                                    |
  |    +----------------------+    |                                    |
  |    |      s_mounts        | ---+               +--------------------+
  |    |                      |                    |
  |    |                      |                    |
  |    |                      |                    |
  |    |                      |                    |
  |    |                      |                    |
  |    |                      |                    V
  |    |                      |       +----------------------+
  |    |       s_root         | ----> |       dentry /       | <--------dh------ dentry_hashtable
  |    |       ......         |       +----------------------+
  |    +----------------------+       | d_name = {hash, ''}  |
  |         ^                         | d_parent             |
  |         |                         | d_inode = inode_/(2) |
  |         |                         | d_sb                 |
  |         |                         | d_child              |
  |         |    +------------------- | d_subdirs            |
  |         |    |                    +----------------------+
  |         |    |                               |
  |         |    |                      +---dh---+
  |         |    |                      V
  |         |    |         +--------------------------+    +------------------------+     +------------------------+
  |         |    |         |       dentry a           |-dh>|       dentry usr       | --> |       dentry var       | ---+
  |         |    |         +--------------------------+    +------------------------+     +------------------------+    |
  |         |    |         | d_name = {hash, 'a'}     |    | d_name = {hash, 'usr'} |     | d_name = {hash, 'var'} |    |
  |         |    |         | d_parent = dentry /      |    | d_parent = dentry /    |     | d_parent = dentry /    |    |
  |         |    |         | d_inode = inode_a        |    | d_inode = inode_usr    |     | d_inode = inode_var    |    |
  |         +------------- | d_sb                     |    | d_sb                   |     | d_sb                   |    d
  |              +-------> | d_child                  |--> | d_child                | --> | d_child                |    h
  |                        | d_subdirs                |-+  | d_subdirs              |     | d_subdirs              |    |
  |                        | d_flags |= DCACHE_MOUNTED| |  +------------------------+     +------------------------+    |
  |                        +--------------------------+ |                                                               |
  |                                  ^                  |                                                               |
  |                                  |                  |                                                               |
  |                                  |                  |  +----------------------+       +----------------------+      |
  |                                  |          +--------- |       dentry b       | <-dh- |       dentry x       | <----+
  |                                  |          |       |  +----------------------+       +----------------------+
  |                                  |          |       |  | d_name = {hash, 'b'} |       | d_name = {hash, 'x'} |
  |                                  |          |       |  | d_parent = dentry a  |       | d_parent = dentry a  |
  |                                  |          |       |  | d_inode = inode_b    |       | d_inode = inode_x    |
  |                                  |          |       |  | d_sb                 |       | d_sb                 |
  |                                  |          |       +->| d_child              | ----> | d_child              |
  |                                  |          |          | d_subdirs            | --+   | d_subdirs            |
  |                                  |          |          +----------------------+   |   +----------------------+
  |                                  |          |                                     |
  |                                  |          |                                     |
  |                                  |          |                                     |     +----------------------+         +----------------------+
  |                                  |          +-------------------dh--------------------> |       dentry c       | --dh--> |       dentry y       |
  |                                  |                                                |     +----------------------+         +----------------------+
  |                                  |                                                |     | d_name = {hash, 'c'} |         | d_name = {hash, 'y'} |
  |                                  |                                                |     | d_parent = dentry b  |         | d_parent = dentry b  |
  |                                  |                                                |     | d_inode = inode_c    |         | d_inode = inode_y    |
  |                                  |                                                |     | d_sb                 |         | d_sb                 |
  |                                  |                                                +---> | d_child              |  ---->  | d_child              |
  |                                  |                                                      | d_subdirs            |         | d_subdirs            |
  |                                  |                                                      +----------------------+         +----------------------+
  |                                  |
  |                                  |
  |                                  |
  |                                  |
  |                                  |
  |                                  |
  |                                  |
  |                                  |    +-------------------------+
  |                                  |    |     struct mount        |
  |                                  |    +-------------------------+
  |                                  |    | +---------------------+ |
  |                                  |    | | struct vfsmount mnt | |
  |                                  |    | +---------------------+ |
  |                                  |    | |       mnt_root      |-|--+
  |                 +---------------------|-|        mnt_sb       | |  |
  |                 |                |    | +---------------------+ |  |
  |                 |                |    |                         |  |
  |                 |             +------>|      mnt_instance       |  |
  |                 |             |  |    |                         |  |
  |                 |             |  +----|       mnt_mountpoint    |  |
  |                 |             |       |                         |  |
  +---------------------------------------|       mnt_parent        |  |
                    |             |       |                         |  |
                    |             |       +-------------------------+  |
                    |             |                                    |
                    |             |                                    |
                    V             |                                    |
      +----------------------+    |                                    |
      |       superblock     |    |                                    |
      +----------------------+    |                                    |
      |      s_mounts        | ---+               +--------------------+
      |                      |                    |
      |                      |                    |
      |                      |                    |
      |                      |                    |
      |                      |                    |
      |                      |                    V
      |                      |       +----------------------+
      |       s_root         | ----> |       dentry /       |
      |       s_bdev         |       +----------------------+
      +----------------------+       | d_name = {hash, ''}  |
                                     | d_parent             |
                                     | d_inode = inode_/    |
                                     | d_sb                 |
                                     | d_child              |
                                     | d_subdirs            |
                                     +----------------------+

```