import os
import sys
import subprocess
#######################################################################
# HELPER FUNCTION
def getEnvVar(name, default):
   if (os.environ.has_key(name)):
      return os.environ[name]
   else:
      return default
#######################################################################

# Gets the environment variables set by the user
env = Environment(ENV = os.environ)
envPlugin = Environment(ENV = os.environ)

env.Append(CCFLAGS = '-std=c++11')
env.Append(LIBS = ['m', 'dl'])
env.Append(LINKFLAGS = '-rdynamic')

envPlugin.Append(SHCCFLAGS = '-I'+os.environ['PWD'])
envPlugin.Append(SHCCFLAGS = '-std=c++11')
envPlugin.Append(LIBS = ['gsl'])

# Language arguments
# For now I am assuming 1 UNLESS they specify 0
# Note we have 'pluma.cpp' and scons, so I must assume they have C++
# and Python
docuda = ARGUMENTS.get('cuda', 1)
r = ARGUMENTS.get('r', 1)
perl = ARGUMENTS.get('perl', 1)

###################################################################
# PYTHON INFORMATION
python_dir = sys.exec_prefix
python_version = sys.version[0:3]
python_include = '-I'+python_dir+'/include/python'+python_version
python_lib = python_dir+'lib/python'+python_version+'/config'
env.Append(CCFLAGS = python_include)
env.Append(LIBPATH = [python_lib])
env.Append(LIBS = ['python'+python_version])
###################################################################

###################################################################
# R (IF INSTALLED)
if (r==1):
   # Get the installation directory of R
   # Note: Will assume /usr/share/R if RHOME is unset
   #rhome = getEnvVar('RHOME', '/usr/share/R')
   rhome = getEnvVar('RHOME', '/usr/local/lib/R/')
   # Get the installation directory of R site-library
   # Note: Will assume /usr/local/lib/R/site-library if RSITELIBHOME is unset
   rsite = getEnvVar('RSITELIBHOME', '/usr/local/lib/R/library')
 
   env.Append(CCFLAGS = '-I'+rsite+'/RInside/include')
   env.Append(CCFLAGS = '-I'+rsite+'/Rcpp/include')
   env.Append(CCFLAGS = '-I'+rhome+'/include')
   env.Append(CCFLAGS = '-DHAVE_R')
   env.Append(LIBPATH = [rsite+'/RInside/lib'])
   env.Append(LIBPATH = [rsite+'/Rcpp/libs'])
   env.Append(LIBPATH = [rhome+'/lib'])
   env.Append(LIBS = ['R'])
   env.Append(LIBS = ['RInside'])
###################################################################

###################################################################
if (docuda==1):
   envPluginCUDA = Environment(ENV = os.environ)
   envPluginCUDA.Tool('cuda')
   envPluginCUDA2 = Environment(ENV = os.environ)
   envPluginCUDA.Tool('cuda')
   envPluginCUDA.Append(NVCCFLAGS = '-I'+os.environ['PWD'])
   envPluginCUDA.Append(NVCCFLAGS = ['-arch=sm_30'])
   envPluginCUDA.Append(NVCCFLAGS = ['--ptxas-options=-v']) 
   envPluginCUDA.Append(NVCCFLAGS = ['-std=c++11'])
   envPluginCUDA.Append(NVCCFLAGS = ['-Xcompiler'])
   envPluginCUDA.Append(NVCCFLAGS = ['-fpic'])
   envPluginCUDA2['NVCCFLAGS'] = ['-shared']
###################################################################

###################################################################
if (perl==1):
   # Get the installation directory of Perl
   # Note: Will assume /usr/ if PERLHOME is unset
   # Also need pthreads, assuming /usr/local is PTHREADHOME is unset
   perlhome = getEnvVar('PERLHOME', '/usr/')
   pthreadhome = getEnvVar('PTHREADHOME', '/usr/local/')
   getversion = subprocess.check_output(['perl', '-V']).split(' ')
   perlversion = getversion[5]+'.'+getversion[7]

   env.Append(CCFLAGS = '-DHAVE_PERL')
   env.Append(LIBPATH = [perlhome+'/lib'])
   env.Append(CCFLAGS = '-Wl,-E')
   env.Append(CCFLAGS = '-fstack-protector')
   env.Append(LIBPATH = [perlhome+'/lib'])
   env.Append(LIBPATH = [perlhome+'/perl/'+perlversion+'/CORE'])
   env.Append(LIBPATH = [pthreadhome+'/lib/'])
   env.Append(LIBS = ['perl'])
   env.Append(LIBS = ['dl'])
   env.Append(LIBS = ['m'])
   env.Append(LIBS = ['pthread'])
   env.Append(LIBS = ['c'])
   env.Append(LIBS = ['crypt'])
   env.Append(CCFLAGS = '-DREENTRANT')
   env.Append(CCFLAGS = '-D_GNU_SOURCE')
   env.Append(CCFLAGS = '-DDEBIAN')
   env.Append(CCFLAGS = '-fstack-protector')
   env.Append(CCFLAGS = '-Wno-literal-suffix')
   env.Append(CCFLAGS = '-fno-strict-aliasing')
   env.Append(CCFLAGS = '-pipe')
   env.Append(CCFLAGS = '-I'+pthreadhome+'/include/')
   env.Append(CCFLAGS = '-DLARGEFILE_SOURCE')
   env.Append(CCFLAGS = '-D_FILE_OFFSET_BITS=64')
   env.Append(CCFLAGS = '-I'+perlhome+'/lib/perl/'+perlversion+'/CORE')

###################################################################
# Finally, compile!

#####################################################
# Assemble plugin path
####################################################
#pluginpath = ['plugins2/']
pluginpath = ['plugins/']
miamipluginpath = getEnvVar('PLUMA_PLUGIN_PATH', '')
if (miamipluginpath != ''):
   pluginpaths = miamipluginpath.split(':') #+ ":" + pluginpath
   pluginpath += pluginpaths
print "PLUGIN DIRECTORIES: ", pluginpath
######################################################
# C++ Plugins
for folder in pluginpath:
 #pluginlist_cpp = Glob('plugins/*/*.cpp')
 pluginlist_cpp = Glob(folder+'/*/*.cpp')
 for plugin in pluginlist_cpp:
   print plugin
   x = envPlugin.SharedLibrary(source=plugin)
   print x
######################################################
######################################################
# CUDA Plugins
if (docuda==1):
 for folder in pluginpath:
   pluginlist_cu = Glob(folder+'/*/*.cu')
   for plugin in pluginlist_cu:
      envPluginCUDA.Object(source=plugin)
      pluginname = plugin.path[0:len(plugin.path)-3]
      pluginfile = plugin.name[0:len(plugin.name)-3]
      sharedpluginname = pluginname.replace(pluginfile, 'lib'+pluginfile)
      envPluginCUDA.Command(sharedpluginname+".so", pluginname+".o", "nvcc -o $TARGET -shared $SOURCE")
#####################################################

# Main Executable
folder_sets = [['.', 'languages'], ['PluGen']] 
sources = [[], []]
targets = ['pluma', 'PluGen/PluGen']

for i in range(0,len(targets)):
   for folder in folder_sets[i]:
      env.Append(CCFLAGS = '-I'+folder)
      sources[i].append(Glob(folder+'/*.cpp'))
   print sources[i], targets[i]
   env.Program(source=sources[i], target=targets[i])   

#for folder in folders:
#   env.Append(CCFLAGS = '-I'+folder)
#   sources.append(Glob(folder+'/*.cpp'))
#env.Program(source=sources, target='miami')
#env.Append(CCFLAGS = '-I.')
#env.Append(CCFLAGS = '-Ilanguages')

#env.Program(source=['miami.cpp', Glob('languages/*.cpp')], target='miami')

###################################################################


