import { Box, Button, Typography, useTheme } from "@mui/material";
import { useEffect, useState } from "react";
import { svg } from "@/components/Svg";
import Link from "next/link";
import { useRouter } from "next/router";
import { useIntl } from "react-intl";
import { axiosGet } from "@/components/axiosCall";
import { useSelector } from "react-redux";
import PagnationTable from "@/components/PagnationTable";
import GetTimeinTable from "@/components/GetTimeinTable";
import ImageLoad from "@/components/ImageLoad";
import { getUrlPageName } from "@/Share/_helper";
import { toast } from "react-toastify";
import SendMoneyModal from "@/components/SendMoneyModal";
import WithdrawalModal from "@/components/WithdrawalModal";

function Setting() {
  const theme = useTheme();
  const { locale } = useRouter();
  const { messages } = useIntl();
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(false)
  const user = useSelector(rx => rx.auth.data)
  const [paginationModel, setPaginationModel] = useState({ page: 0, pageSize: 10 })
  const [totalRows, setTotalRows] = useState(0)
  const [balance, setBalance] = useState([])
  const [loading2, setLoading2] = useState(false)
  const [open, setOpen] = useState(false);
  const [refetch, setRefetch] = useState(0)

  useEffect(() => {
    if (user) {
      axiosGet("auth/users/get-user-money-report/")
        .then((res) => {
          if (res.status) {
            setData(res.results)
          }
        }).finally(() => {
          setLoading(true)
        })
    }
  }, [user, paginationModel, locale, refetch])
  useEffect(() => {
    if (user) {
      const loading = toast.loading(locale === "ar" ? "جارى تحميل..." : "Loading...")
      setLoading2(false)
      setBalance([])
      axiosGet(`auth/money-report/?user__id=${user.id}&page=${paginationModel.page + 1}&page_limit=${paginationModel.pageSize}`, locale)
        .then((res2) => {
          if (res2.status) {
            setTotalRows(res2.count)
            setBalance(res2.results)

          }
        }).finally(() => {
          setLoading2(true)
          toast.done(loading)
        })
    }
  }, [user, paginationModel, locale, refetch])



  const InvitationsColumns = [
    {
      flex: 0.2,
      minWidth: 50,
      field: 'index',
      headerName: '#',
      renderCell: params => {
        const { row } = params

        return (
          <Box sx={{ display: 'flex', alignItems: 'center' }} className="!py-2">
            <Box sx={{ display: 'flex', flexDirection: 'column' }}>
              <Typography variant='body2' sx={{ color: 'text.primary', fontWeight: 600 }}>
                {row.index + 1}
              </Typography>
            </Box>
          </Box>
        )
      }
    },

    {
      flex: 0.9,
      minWidth: 270,
      field: 'user_data.first_name',
      headerName: locale === 'ar' ? 'العملية' : 'Operation',
      renderCell: params => {
        const { row } = params

        return (
          <div className="!py-2">
            {(row.description.includes("binance_id")) && <>{locale === "ar" ? "اضافة رصيد عن طريق Binance" : "Add Balance By Binance"}</>}
            {(row.tool) &&
              <Link href={`/${locale}/tool-rental/product/${getUrlPageName(row.tool_info.server_info.name)}*${row.tool_info.server_info.id}`} className="flex || items-center || gap-2">
                <div className="!w-[50px]  !h-[50px] rounded-full min-w-[50px]">
                  <ImageLoad src={row.tool_info.server_info.image} className="object-cover h-full rounded-full" alt={row.tool_info.server_info.name} />

                </div>
                <div>
                  <h2 className="text-sm || font-semibold">{row.tool_info.server_info.name}</h2>
                </div>
              </Link>
            }
            {(row.file) &&
              <Link href={`/${locale}/file-selling/view/${getUrlPageName(row.file_info.server_info.name)}*${row.file_info.server_info.id}`} className="flex || items-center || gap-2">
                <div className="!w-[50px]  !h-[50px] rounded-full min-w-[50px]">
                  <ImageLoad src={row.file_info.server_info.image} className="object-cover h-full rounded-full" alt={row.file_info.server_info.name} />
                </div>
                <div>
                  <h2 className="text-sm || font-semibold">{row.file_info.server_info.name}</h2>
                </div>
              </Link>
            }
            {(row.problem_solve) &&
              <Link href={`/${locale}/problem-solving/view/${getUrlPageName(row.problem_solve_info.server_info.name)}*${row.problem_solve_info.server_info.id}`} className="flex || items-center || gap-2">
                <div className="!w-[50px]  !h-[50px] rounded-full min-w-[50px]">
                  <ImageLoad src={row.problem_solve_info.server_info.image} className="object-cover h-full rounded-full" alt={row.problem_solve_info.server_info.name} />
                </div>
                <div>
                  <h2 className="text-sm || font-semibold">{row.problem_solve_info.server_info.name}</h2>
                </div>
              </Link>
            }
            {(row.creator_info && row.kind === "positive") &&
              <>{locale === "ar" ? "اضافة رصيد عن طريق المشرف" : "Add Balance By Supervisor"}</>
            }
            {(row.creator_info && row.kind === "negative") &&
              <>{locale === "ar" ? "طرح رصيد عن طريق المشرف" : "Subtract Balance By Supervisor"}</>
            }
          </div>
        )
      }
    },

    {
      flex: 0.9,
      minWidth: 270,
      field: 'User',
      headerName: locale === 'ar' ? 'عن طريق' : 'By',
      renderCell: params => {
        const { row } = params

        return (
          <Box sx={{ display: 'flex', alignItems: 'center', width: "100%" }} className="!py-2">
            {row.kind == "positive" ?
              <>

                {(row.tool) &&
                  <Link href={`/${locale}/profile/${row.tool_info.user_info.id}`} className="flex || items-center || gap-2">
                    <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                      <ImageLoad src={row.tool_info.user_info.image} className="object-cover h-full rounded-full" alt={row.tool_info.user_info.first_name} />
                    </div>
                    <div>
                      <h2 className="text-sm || font-semibold capitalize">{row.tool_info.user_info.first_name} {row.tool_info.user_info.last_name}</h2>
                      <h2 className="text-sm || font-semibold">{row.tool_info.user_info.email}</h2>
                      <h2 className="text-sm || font-semibold">#{row.tool_info.user_info.id}</h2>
                    </div>
                  </Link>
                }
                {(row.problem_solve) &&
                  <Link href={`/${locale}/profile/${row.problem_solve_info.user_info.id}`} className="flex || items-center || gap-2">
                    <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                      <ImageLoad src={row.problem_solve_info.user_info.image} className="object-cover h-full rounded-full" alt={row.problem_solve_info.user_info.first_name} />
                    </div>
                    <div>
                      <h2 className="text-sm || font-semibold capitalize">{row.problem_solve_info.user_info.first_name} {row.problem_solve_info.user_info.last_name}</h2>
                      <h2 className="text-sm || font-semibold">{row.problem_solve_info.user_info.email}</h2>
                      <h2 className="text-sm || font-semibold">#{row.problem_solve_info.user_info.id}</h2>
                    </div>
                  </Link>
                }
                {(row.file) &&
                  <Link href={`/${locale}/profile/${row.file_info.user_info.id}`} className="flex || items-center || gap-2">
                    <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                      <ImageLoad src={row.file_info.user_info.image} className="object-cover h-full rounded-full" alt={row.file_info.user_info.first_name} />
                    </div>
                    <div>
                      <h2 className="text-sm || font-semibold capitalize">{row.file_info.user_info.first_name} {row.file_info.user_info.last_name}</h2>
                      <h2 className="text-sm || font-semibold">{row.file_info.user_info.email}</h2>
                      <h2 className="text-sm || font-semibold">#{row.file_info.user_info.id}</h2>
                    </div>
                  </Link>
                }
              </>
              :
              <>
                <>

                  {(row.tool) &&
                    <Link href={`/${locale}/profile/${row.tool_info.server_info.user_data.id}`} className="flex || items-center || gap-2">
                      <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                        <ImageLoad src={row.tool_info.server_info.user_data.image} className="object-cover h-full rounded-full" alt={row.tool_info.server_info.user_data.first_name} />
                      </div>
                      <div>
                        <h2 className="text-sm || font-semibold capitalize">{row.tool_info.server_info.user_data.first_name} {row.tool_info.server_info.user_data.last_name}</h2>
                        <h2 className="text-sm || font-semibold">{row.tool_info.server_info.user_data.email}</h2>
                        <h2 className="text-sm || font-semibold">#{row.tool_info.server_info.user_data.id}</h2>
                      </div>
                    </Link>
                  }
                  {(row.problem_solve) &&
                    <Link href={`/${locale}/profile/${row.problem_solve_info.server_info.user_date.id}`} className="flex || items-center || gap-2">
                      <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                        <ImageLoad src={row.problem_solve_info.server_info.user_date.image} className="object-cover h-full rounded-full" alt={row.problem_solve_info.server_info.user_date.first_name} />
                      </div>
                      <div>
                        <h2 className="text-sm || font-semibold capitalize">{row.problem_solve_info.server_info.user_date.first_name} {row.problem_solve_info.server_info.user_date.last_name}</h2>
                        <h2 className="text-sm || font-semibold">{row.problem_solve_info.server_info.user_date.email}</h2>
                        <h2 className="text-sm || font-semibold">#{row.problem_solve_info.server_info.user_date.id}</h2>
                      </div>
                    </Link>
                  }
                  {(row.file) &&
                    <Link href={`/${locale}/profile/${row.file_info.server_info.user_data.id}`} className="flex || items-center || gap-2">
                      <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                        <ImageLoad src={row.file_info.server_info.user_data.image} className="object-cover h-full rounded-full" alt={row.file_info.server_info.user_data.first_name} />
                      </div>
                      <div>
                        <h2 className="text-sm || font-semibold capitalize">{row.file_info.server_info.user_data.first_name} {row.file_info.server_info.user_data.last_name}</h2>
                        <h2 className="text-sm || font-semibold">{row.file_info.server_info.user_data.email}</h2>
                        <h2 className="text-sm || font-semibold">#{row.file_info.server_info.user_data.id}</h2>
                      </div>
                    </Link>
                  }
                </>
              </>
            }

            {(row.creator_info) &&
              <Link href={`/${locale}/profile/${row.creator_info.id}`} className="flex || items-center || gap-2">
                <div className="!w-[40px]  !h-[40px] rounded-full min-w-[40px]">
                  <ImageLoad src={row.creator_info.image} className="object-cover h-full rounded-full" alt={row.creator_info.first_name} />
                </div>
                <div>
                  <h2 className="text-sm || font-semibold capitalize">{row.creator_info.first_name} {row.creator_info.last_name}</h2>
                  <h2 className="text-sm || font-semibold">{row.creator_info.email}</h2>
                  <h2 className="text-sm || font-semibold">#{row.creator_info.id}</h2>
                </div>
              </Link>
            }
            {(row.description.includes("binance_id")) && <div className="w-full text-2xl text-center">
              <div className="w-[40px] mx-auto" dangerouslySetInnerHTML={{ __html: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 126.61 126.61"><g fill="#f3ba2f"><path d="m38.73 53.2 24.59-24.58 24.6 24.6 14.3-14.31-38.9-38.91-38.9 38.9z"/><path d="m0 63.31 14.3-14.31 14.31 14.31-14.31 14.3z"/><path d="m38.73 73.41 24.59 24.59 24.6-24.6 14.31 14.29-38.9 38.91-38.91-38.88z"/><path d="m98 63.31 14.3-14.31 14.31 14.3-14.31 14.32z"/><path d="m77.83 63.3-14.51-14.52-10.73 10.73-1.24 1.23-2.54 2.54 14.51 14.5 14.51-14.47z"/></g></svg>` }}></div>
            </div>}

          </Box>
        )
      }
    },


    {
      flex: 0.2,
      minWidth: 100,
      field: 'amount',
      sortable: false,
      headerName: locale === "ar" ? "المبلغ" : "Amount",
      renderCell: params => (
        <Box sx={{ display: 'flex', alignItems: 'center', gap: 2 }} className="!py-2">
          <p className={`${params.row.kind === "positive" ? "text-green-500" : "text-red-500"}`}><span className="text-[20px]">{params.row.kind === "positive" ? "+" : "-"} </span> ${params.row.money}</p>
        </Box>
      )
    }, {
      flex: 0.2,
      minWidth: 200,
      field: 'updated',
      sortable: false,
      headerName: locale === "ar" ? "التاريخ" : "Date",
      renderCell: params => (
        <Box sx={{ display: 'flex', alignItems: 'center', gap: 2 }} className="!py-2">
          <GetTimeinTable data={params.row.created} />
        </Box>
      )
    },


  ]
  const [withdrawalOpen, setWithdrawalOpen] = useState(false)

  return (
    <>
      <SendMoneyModal open={open} setOpen={setOpen} setRefetch={setRefetch}   setPaginationModelData={setPaginationModel}/>
      <WithdrawalModal open={withdrawalOpen} setOpen={setWithdrawalOpen} dataCall={data} />
      <div
        style={{ boxShadow: theme.shadows[1] }}
        className="pb-5 || bg-white || rounded-md"
      >
        <div className="pt-5 || pb-3 || mx-5 || border-b || border-gray-200 || flex || flex-wrap || justify-between || gap-3">
          <div className="flex || items-center || gap-2 || flex-wrap">
            <div className="w-[25px]">{svg.money}</div>
            <h2 className="font-semibold">{messages.balance}</h2>
          </div>
          <div className="flex || gap-2 || md:flex-row || flex-col w-full md:w-auto ">


            <Button
              onClick={() => setOpen(true)}
              variant="contained"
              className="!font-semibold"
              color="warning"
            >
              {locale === "ar" ? "أرسال رصيد للمستخدم" : "Send Balance to user"}
            </Button>
            <Button
              onClick={() => setWithdrawalOpen(true)}
              variant="contained"
              className="!font-semibold"
              color="secondary"
            >
              {locale === "ar" ? "طلب سحب الرصيد" : "Withdrawal Request"}
            </Button>
            <Button
              LinkComponent={Link}
              href={`/${locale}/setting/balance/add`}
              variant="contained"
              className="!font-semibold"
            >
              {locale === "ar" ? "شحن رصيد" : "Add Balance"}
            </Button>

          </div>
        </div>
        {loading ?
          <div className="grid || grid-cols-3 || gap-3 ||   || mt-5">
            <div className="|| text-center || text-black/70 || relative flex || flex-col">
              <h2 className="mb-5 || text-[13px] sm:text-base md:text-xl || font-semibold">{locale === "ar" ? "مجموع الارباح" : "Total Profit"}</h2>
              <p className="font-semibold mt-auto || text-sm sm:text-xl md:text-2xl">$ {data.total_profit.toFixed(2)}</p>
              <div className="w-[1px] || flex || items-center || justify-center  absolute || top-1/2 || end-0 h-[50px] || -translate-y-1/2">
                <div className="w-full h-[70%] bg-gray-300"></div>
              </div>
            </div>

            <div className="|| text-center || text-mainColor || relative flex || flex-col">
              <h2 className="mb-5 || text-[13px] sm:text-base md:text-xl || font-semibold">{locale === "ar" ? "اجمالى الرصيد" : "Total Balance"}</h2>
              <p className="font-semibold mt-auto || text-sm sm:text-xl md:text-2xl">$ {data.total_balance.toFixed(2)}</p>
              <div className="w-[1px] || flex || items-center || justify-center  absolute || top-1/2 || end-0 h-[50px] || -translate-y-1/2">
                <div className="w-full h-[70%] bg-gray-300"></div>
              </div>
            </div>

            <div className="|| text-center || text-black/70 flex || flex-col">
              <h2 className="mb-5 || text-[13px] sm:text-base md:text-xl || font-semibold">
                {locale === "ar" ? "الرصيد القابل للسحب" : "Withdrawable Balance"}
              </h2>
              <p className="font-semibold mt-auto || text-sm sm:text-xl md:text-2xl">$ {data.withdrawable_balance.toFixed(2)}</p>
            </div>
          </div>
          :
          <>
            <div className="flex || items-center || justify-center || min-h-[300px]">
              <div className="h-[40px] w-[40px] border-2 border-mainColor border-t-transparent || rounded-full || animate-spin"></div>
            </div>
          </>
        }

        <div className="mt-5"></div>
        <PagnationTable
          Invitationscolumns={InvitationsColumns}
          data={balance?.map((ele, i) => {
            const fData = { ...ele }
            fData.index = i + (paginationModel.page * paginationModel.pageSize)

            return fData
          })}
          totalRows={totalRows}
          getRowId={row => row.id}
          loading={!loading2}
          locale={locale}
          noRow={locale === 'ar' ? 'لا يوجد' : 'Not Found'}
          paginationModel={paginationModel}
          setPaginationModel={setPaginationModel}
          getRowClassName={({ row }) => row.kind === "positive" ? "bg-green-400/20 " : "bg-red-400/20 "}
        />
      </div>

    </>
  );
}

export default Setting;
