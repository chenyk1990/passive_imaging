from rsf.proj import *
#### COPYRIGHT: Chen et al., 2021, SRL
# 
# Reference:
# Chen, Y., O.M. Saad, M. Bai, X. Liu, and S. Fomel, 2021, A compact program for 3D passive seismic source-location imaging, Seismological Research Letters, in press.
  
#### Part I: Specifying parameters ####
nt=1501		#number of samples
dt=0.001	#temporal sampling
nb=30		#size of ABC layers
ct=0.01		#ABC parameter
jsnap=4		#output wavefield interval
ng=4 		#number of groups
nz=141		#samples in Z
nx=141		#samples in X
ny=141 		#samples in Y
dz=20		#sampling in Z
dx=20		#sampling in X
dy=20		#sampling in Y
ic2=1		#squared cross-correlation IC

#for synthetic test
ns=3
sz='50,70,90'
sx='50,60,70'
sy='50,60,70'
f='10,10,10'
t='0.2,0.35,0.5'
A='1,2,2'

#### Part II: Compiling the program ####
exe=Program('mod3d.c')

#### Part III: Creating/Inputing the velocity model ####
Flow('vel',None,'spike n1=%d n2=%d n3=%d d1=%g d2=%g d3=%g mag=4600 | math output="1500+1.2*x1"'%(nz,nx,ny,dz,dx,dy))
## predefining source locatons, plotting the sources onto the velocity model
Flow('src',None,
     '''
     spike n1=%d n2=%d n3=%d d1=%g d2=%g d3=%g nsp=%d
     k1=%s
     k2=%s
     k3=%s
     mag=200000 | smooth rect1=4 rect2=4 rect3=4 repeat=1 
     '''%(nz,nx,ny,dz,dx,dy,ns,sz,sx,sy))
Flow('sov','vel src','add mode=a ${SOURCES[1]} ')
## plotting the velocity models with highlights on source locations
Result('vel1','vel src','add mode=a ${SOURCES[1]} |byte bar=bar.rsf mean=y|grey3  flat=n allpos=y bias=1500 color=j scalebar=n maxval=2200 title="Source Location and Velocity Model" barlabel="V" barunit="m/s" label1=Depth label2="Distance in X" label3="Distance in Y" unit1=m unit2=m unit3=m frame1=49 frame2=49 frame3=49 scalebar=y')
Result('vel2','vel src','add mode=a ${SOURCES[1]} |byte bar=bar.rsf mean=y|grey3  flat=n allpos=y bias=1500 color=j scalebar=n maxval=2200 title="Source Location and Velocity Model" barlabel="V" barunit="m/s" label1=Depth label2="Distance in X" label3="Distance in Y" unit1=m unit2=m unit3=m frame1=69 frame2=59 frame3=59 scalebar=y')
Result('vel3','vel src','add mode=a ${SOURCES[1]} |byte bar=bar.rsf mean=y|grey3  flat=n allpos=y bias=1500 color=j scalebar=n maxval=2200 title="Source Location and Velocity Model" barlabel="V" barunit="m/s" label1=Depth label2="Distance in X" label3="Distance in Y" unit1=m unit2=m unit3=m frame1=89 frame2=69 frame3=69 scalebar=y')

#### Part IV: Generating/Inputing the recorded passive seismic data ####
Flow('data0','vel %s'%exe[0],
     '''
     ./${SOURCES[1]} verb=y cmplx=n ps=y nt=%d dt=%g jsnap=1 abc=y nbt=%d ct=%g src=0 ns=%d
     spz=%s
     spx=%s
     spy=%s
     f0=%s
     t0=%s
     A=%s
     '''%(nt,dt,nb,ct,ns,sz,sx,sy,f,t,A))
## add noise and sub samples
Flow('data','data0','noise var=0.001 type=y seed=12005|window j2=1 j3=1')
Result('data','data','byte | window max1=1.3 |grey3 flat=n frame1=300 frame2=30 frame3=10 clip=1.0 title="Synthetic data" unit2=m')

#### Part V: Backward propagating of the grouped receiver wavefield ####
dg=(int)(nx-nb*2)/ng
print('Group interval is',dg)
snaps_list = []
src_list = ''
for i in range(ng):
    mask = 'mask%d' %i
    data = 'data_mask%d' %i
    img = 'img%d' %i
    snaps = 'snaps%d' %i
    
    Flow(mask,None,'spike n1=%d n2=%d mag=1 k1=%d l1=%d k2=%d l2=%d | sfdd type=int' %(nx-nb*2,ny-nb*2,dg*i+1,dg*i+dg,1,ny-nb*2))
    Flow(data,['data',mask],'headercut mask=${SOURCES[1]}')
    Flow([img,snaps],['vel',data,'%s'%exe[0]],
         '''
         ./${SOURCES[2]} snaps=${TARGETS[1]} verb=y cmplx=n vref=1500 ps=y abc=y nbt=%d ct=%g tri=y dat=${SOURCES[1]} jsnap=%d
         '''%(nb,ct,jsnap))
    Result(snaps,'window j3=10 | grey gainpanel=a')
    Result(img,'grey')
    snaps_list += [snaps]
    src_list += ' ${SOURCES[%d]}' %(i+1)

## view the grouped data
Result('data_mask0','byte | window max1=1.3 |grey3 flat=n frame1=400 frame2=10 frame3=40 clip=0.5 title="Group 1" label2="Distance in X" label3="Distance in Y" unit2=m')
Result('data_mask1','byte | window max1=1.3 |grey3 flat=n frame1=400 frame2=30 frame3=40 clip=0.5 title="Group 2" label2="Distance in X" label3="Distance in Y" unit2=m')
Result('data_mask2','byte | window max1=1.3 |grey3 flat=n frame1=400 frame2=50 frame3=40 clip=0.5 title="Group 3" label2="Distance in X" label3="Distance in Y" unit2=m')
Result('data_mask3','byte | window max1=1.3 |grey3 flat=n frame1=400 frame2=70 frame3=40 clip=0.5 title="Group 4" label2="Distance in X" label3="Distance in Y" unit2=m')

#### Part VI: Applying the cross-correlation imaging condition ####
if ic2:
	if ng==4:
		Flow('ccr0',snaps_list,'math a=${SOURCES[1]} b=${SOURCES[2]} c=${SOURCES[3]} output="input^2*a^2*b^2*c^2"')
	elif ng==6:
		Flow('ccr0',snaps_list,'math a=${SOURCES[1]} b=${SOURCES[2]} c=${SOURCES[3]} d=${SOURCES[4]} e=${SOURCES[5]} output="input^2*a^2*b^2*c^2*d^2*e^2"')
	elif ng==8:
		Flow('ccr0',snaps_list,'math a=${SOURCES[1]} b=${SOURCES[2]} c=${SOURCES[3]} d=${SOURCES[4]} e=${SOURCES[5]} f=${SOURCES[6]} g=${SOURCES[7]} output="input^2*a^2*b^2*c^2*d^2*e^2*f^2*g^2"')
	elif ng==10:
		Flow('ccr0',snaps_list,'math a=${SOURCES[1]} b=${SOURCES[2]} c=${SOURCES[3]} d=${SOURCES[4]} e=${SOURCES[5]} f=${SOURCES[6]} g=${SOURCES[7]} h=${SOURCES[8]} i=${SOURCES[9]} output="input^2*a^2*b^2*c^2*d^2*e^2*f^2*g^2*h^2*i^2"')
else:
	Flow('ccr0',snaps_list,'cat axis=5 ${SOURCES[1:%d]} | stack prod=y axis=5'%len(snaps_list))

Flow('location0','ccr0','stack axis=4 |pad beg1=%d end1=%d beg2=%d end2=%d beg3=%d end3=%d| put o1=0 o2=0 o3=0'%(nb,nb,nb,nb,nb,nb))

#### Part VII: Plotting the source locations ####
Result('location1','location0','threshold1 thr=0.08|byte |grey3 flat=n frame1=50 frame2=50 frame3=49 pclip=99.999999 title="Source Location Image" label1=Depth label2="Distance in X" label3="Distance in Y" unit1=m unit2=m unit3=m')
Result('location2','location0','threshold1 thr=0.08|byte |grey3 flat=n frame1=70 frame2=60 frame3=60 pclip=99.999999 title="Source Location Image" label1=Depth label2="Distance in X" label3="Distance in Y" unit1=m unit2=m unit3=m')
Result('location3','location0','threshold1 thr=0.08|byte |grey3 flat=n frame1=85 frame2=70 frame3=70 pclip=99.999999 title="Source Location Image" label1=Depth label2="Distance in X" label3="Distance in Y" unit1=m unit2=m unit3=m')


End()
